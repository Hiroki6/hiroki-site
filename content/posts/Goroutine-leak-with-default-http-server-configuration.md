---
title: Goroutine leak with default http server configuration
date: "2024-03-03T12:00:00.000Z"
template: "post"
draft: false
slug: "goroutine-leak-with-http-server"
category: "Golang"
tags:
  - "Golang"
description: "How the default http server configuration leads to a goroutine leak"
socialImage: "/media/goroutine-leak/goroutine-top.jpg"
---

I have experienced a goroutine leak once because of a misconfiguration of `http server`.  
In this article, I will explain how it occurred and how to prevent it with a proper configuration by using sample code.

## Setup a server
This server serves as an example and performs the following functions:

1. Accepts a `POST` request and logs the request body at the `/post` endpoint.
2. Provides Prometheus metrics at the `/metrics` endpoint, enabling the observation of basic metrics.
3. Exposes runtime profiling data on port `6060`, allowing for the profiling of runtime data.

```go
// handler function takes the body and log it
func handler(w http.ResponseWriter, r *http.Request) {
	if r.Method != "POST" {
		http.Error(w, "Method is not supported.", http.StatusMethodNotAllowed)
		return
	}

	body, err := io.ReadAll(r.Body)
	if err != nil {
		http.Error(w, "Error reading request body", http.StatusInternalServerError)
		return
	}

	log.Println(string(body))

	defer r.Body.Close()
	w.WriteHeader(http.StatusOK)
	w.Write([]byte("Post request processed."))
}

func main() {
	s := &http.Server{
		Addr:        ":8080"
	}

    // outputs prometheus metrics
	http.Handle("/metrics", promhttp.Handler())
	http.HandleFunc("/post", handler)

	go func() {
		// outputs runtime profilng data at this port
		log.Println(http.ListenAndServe("localhost:6060", nil))
	}()

	if err := s.ListenAndServe(); err != nil {
		log.Fatal("failed to start server", err)
	}
}
```

## Goroutine observation with some scenarios
Let's send some requests by various scenarios and see how goroutines behave.

### Scenarios: 1 Send normal http requests
In the scenario 1, I send some http requests.

The client below sends 30 requests every second for 3 minutes.

```go
func main() {
	url := "http://localhost:8080/post"

	numGoroutines := 30 // Number of parallel requests to send

	for i := 0; i < numGoroutines; i++ {
		go sendRequest(url) // Start a new goroutine for each request
	}

	time.Sleep(3 * time.Minute)
}

func sendRequest(url string) {
	for {
		res, err := http.Post(url, "plain/text", bytes.NewBuffer([]byte("test")))
		if err != nil {
			log.Printf("Error sending request: %v", err)
			continue
		}
		body, err := io.ReadAll(res.Body)
		if err != nil {
			log.Printf("Error reading response body: %v", err)
			res.Body.Close()
			continue
		}
		res.Body.Close() 

		fmt.Printf("Received response: %s\n", body)
		time.Sleep(1 * time.Second) // Throttle the requests (optional)
	}
}
```


After sending requests, I got this metrics below. 

As you can see, the number of goroutines is stable. It's always around 10.

![normal_request](/media/goroutine-leak/normal_request.png)

### Scenario 2: Open connections to the server then wait without sending requests
In the scenario 2, I just open connections to the server, __but wait for 3 minutes without sending requests__.
This client below opens 10 connections every second for 3 minutes.

```go
func main() {
	numGoroutines := 10 // Number of parallel requests to send

	for i := 0; i < 180; i++ {
		for j := 0; j < numGoroutines; j++ {
			go openConnection() // Start a new goroutine for each request
		}
		time.Sleep(1 * time.Second)
	}
}

// openConnection opens a connection to the server and wait for 3 minutes
func openConnection() {
	serverAddress := "127.0.0.1:8080"

	conn, err := net.Dial("tcp", serverAddress)
	if err != nil {
		fmt.Println("Error connecting to server:", err)
		return
	}
	defer conn.Close()

	time.Sleep(3 * time.Minute)
}
```


The below is the metrics after client finished the process.  
As you can see, the number of goroutine were growing as long as the client keeps the connection.

![goroutine-leak](/media/goroutine-leak/goroutine_leak.png)

Ok, somehow goroutine is leaked.  
In order to see where the goroutine is leaked, I dumped the goroutine state by calling the profiling endpoint.
```
curl http://localhost:6060/debug/pprof/goroutine\?debug=1 > goroutine_dump.txt
```

At that point, there were 10005 goroutines and almost all goroutine are generated from the `http server`.
```
goroutine profile: total 10005
10000 @ 0x43d5ae 0x4363b7 0x46ed25 0x4c8e87 0x4c99da 0x4c99c8 0x577b45 0x581ba5 0x6429eb 0x5eec23 0x5ef709 0x5ef965 0x5f25a5 0x63c80e 0x63c829 0x643f88 0x648279 0x474141
#       0x46ed24        internal/poll.runtime_pollWait+0x84             /usr/local/go/src/runtime/netpoll.go:345
#       0x4c8e86        internal/poll.(*pollDesc).wait+0x26             /usr/local/go/src/internal/poll/fd_poll_runtime.go:84
#       0x4c99d9        internal/poll.(*pollDesc).waitRead+0x279        /usr/local/go/src/internal/poll/fd_poll_runtime.go:89
#       0x4c99c7        internal/poll.(*FD).Read+0x267                  /usr/local/go/src/internal/poll/fd_unix.go:164
#       0x577b44        net.(*netFD).Read+0x24                          /usr/local/go/src/net/fd_posix.go:55
#       0x581ba4        net.(*conn).Read+0x44                           /usr/local/go/src/net/net.go:179
#       0x6429ea        net/http.(*connReader).Read+0x14a               /usr/local/go/src/net/http/server.go:789
#       0x5eec22        bufio.(*Reader).fill+0x102                      /usr/local/go/src/bufio/bufio.go:110
#       0x5ef708        bufio.(*Reader).ReadSlice+0x28                  /usr/local/go/src/bufio/bufio.go:376
#       0x5ef964        bufio.(*Reader).ReadLine+0x24                   /usr/local/go/src/bufio/bufio.go:405
#       0x5f25a4        net/textproto.(*Reader).readLineSlice+0xa4      /usr/local/go/src/net/textproto/reader.go:56
#       0x63c80d        net/textproto.(*Reader).ReadLine+0xad           /usr/local/go/src/net/textproto/reader.go:39
#       0x63c828        net/http.readRequest+0xc8                       /usr/local/go/src/net/http/request.go:1059
#       0x643f87        net/http.(*conn).readRequest+0x247              /usr/local/go/src/net/http/server.go:1004
#       0x648278        net/http.(*conn).serve+0x338                    /usr/local/go/src/net/http/server.go:1964

...
```

What was actually happening here?

The below image helps finding the reason.
So, when the client establishes the connection to the server, the server spawns a goroutine and wait for the request.
![timeout_overview](/media/goroutine-leak/timeout_overview.png)

Then, what happens if the client doesn't send a request after established the connection?

There are a couple of timeout configurations in the `http server`.
- ReadTimeout: the maximum duration for reading the entire request, including the body
- ReadHeaderTimeout: the maximum duration allowed to read request headers
- WriteTimeout: the maximum duration before timing out writes of the response. It is reset whenever a new request’s header is read.

Basically, when the server reaches out the configured timeout, it closes the connection and deletes the goroutine waiting for the request.
However, the default timeout is none, which means it waits __forever__.
Therefore, the leaked goroutines stayed until the client stopped the connection.


### Scenario 3: Prevent goroutine leaks
Ok, let's setup a `read timeout` then.

I just setup 5 seconds `read timeout` to the `http server` as below.

```go
func main() {
	s := &http.Server{
		Addr:        ":8080",
		ReadTimeout: 5 * time.Second,
	}

	...
}
```

As you can see, the number of goroutines was not growing.

![readtimeout](/media/goroutine-leak/readtimeout.png)

## Conclusion
As we saw how the default http server configuraion leads to a goroutine leak, which is pretty dangerous.
We must set up timeouts for the `http server`. 

In addition, it applies to the `http client` as well. [This article](https://blog.cloudflare.com/the-complete-guide-to-golang-net-http-timeouts/) explains about the client timeout.

## References
- [Diving into Go's HTTP server timeouts](https://adam-p.ca/blog/2022/01/golang-http-server-timeouts/)
- [100 Go Mistakes and How to Avoid Them](https://www.manning.com/books/100-go-mistakes-and-how-to-avoid-them)
- [The complete guide to Go net/http timeouts](https://blog.cloudflare.com/the-complete-guide-to-golang-net-http-timeouts/)