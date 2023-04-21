---
title: CTFlearn Writeups - Reverse engineering -
date: "2022-05-10T12:00:00.000Z"
template: "post"
draft: false
slug: "writesup-reverse-engineering"
category: "Reverse Engineering"
tags:
  - "CTF"
  - "Security"
  - "Reverse Engineering"
description: "Writeups of 'Ramada' and 'Recklinghausen' in CTFlearn reverse engineering challenge"
socialImage: "/media/ctfwritesup/binary.jpg"
---

![binary](/media/ctfwritesup/binary.jpg)

This article is about writeups of [Ramada](https://ctflearn.com/challenge/1009) and [Recklinghausen](https://ctflearn.com/challenge/995). These challenges are categorized in the `Reverse Engineering` category and the difficulty is `Easy`. The same person created these challenges. `Ramada` is a 10-point challenge and `Recklinghausen` is a 20-point challenge. Therefore `Recklinghausen` is more complicated than `Ramada` as described in the description of `Recklinghausen`.

>> This is the 4th in a series of Beginner Reversing Challenges. If you are new to Reversing, you may want to solve Reykjavik, then Riyadh then Rangoon before solving this challenge. This is a 20 point challenge and is different in two ways from the previous 10 point challenges.

- [Ramada Writeup](#ramada-writeup)
- [Recklinghausen Writesup](#recklinghausen-writesup)

## Ramada Writesup

This program requires the flag as an argument. The flag has to be `CTFlearn{kernel}`. I need to find the proper `kernel` value.
```
└─[0] <> ./Ramada
Welcome to the CTFLearn Ramada Reversing Challenge!
        pid :      30858 0x0000788a
        ppid:      30361 0x00007699
Usage: Ramada CTFlearn{kernel}

└─[1] <> ./Ramada CTFlearn{xxxxx}
Welcome to the CTFLearn Ramada Reversing Challenge!
        pid :      31331 0x00007a63
        ppid:      30361 0x00007699
Your flag is the wrong length dude!
```

### static analysis with r2

I start doing a static analysis with r2 to see a function graph.

```
└─[0] <> r2 Ramada
[0x00001280]> aaa
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze function calls (aac)
...

# list all functions
[0x00001280]> afl
...

[0x00001280]> s main
[0x00001100]> VV
```

#### Flag format validation

At first, the program checks if the input starts with `CTFlearn`.

```s
mov ecx, 9
lea rsi, str.CTFlearn
mov rdi, rbp
repe cmpsb byte [rsi], byte ptr [rdi]
seta r12b
sbb r12b, 0
movsx r12d, r12b
test r12d, r12d
jne 0x125d
```

Then, it also checks the input ends with `}`.
```
mov rdi, rbp
call sym.imp.strlen
cmp byte [rbp + rax - 1], 0x7d ; 0x7d = '}'
jne 0x121f
```

#### Flag size

Second, the program checks the input size. The flag size must be `0x1f`, which means __31__.
```
cmp rax, 0x1f
jne 0x120b
```

#### Flag value validation

After that, the program calls the `CheckFlag` function which checks the flag value. I visualize the `CheckFlag` function.
```
[0x00001280]> s sym.CheckFlag_char_const_
[0x00001370]> VV
```

There is a loop that checks each character of the flag. Based on the code below, I get this statement.
`flag**3 = dword [$rsi+$rax*4]` 

```
movsx ecx, byte [rdi + rax] ; each char of the flag(v) in the loop
mov edx, ecx ; edx = v
imul edx, ecx ; edx *= v
imul edx, ecx ; edx *= v
cmp dword [rsi + rax*4], edx
je 0x1380
```

![ramada_check_flag](/media/ctfwritesup/ramada_check_flag.png)

As the result of the static analysis, I get the following these two things.
- The flag size is `31`
- The flag value in the loop is `flag**3 = dword [$rsi+$rax*4]`

### exploit with gdb
Now I exploit the flag dynamically with a Python script and GDB. 
Here is the Python script.
```python
import gdb
import math
# setting a breakpoint where the program checks the flag value. 
gdb.execute('break *0x555555555396')
FLAG_SIZE = 21
dummy_flag = 'x' * FLAG_SIZE
gdb.execute('run CTFlearn{{{}}}'.format(dummy_flag))

data = []

ans = "CTFlearn{"

def find_cube_root(v):
    return math.ceil(math.pow(v, 1/3))

for i in range(21):
    offset = i * 4
    data = gdb.execute('x/xw $rsi+{}'.format(offset), to_string=True)
    v = data.replace("\n", "").split('\t')[1]
    c = chr(find_cube_root(int(v, 0)))
    ans += c

ans += "}"
print(ans)
```

I get the flag by running the script with GDB.
```shell
gdb Ramada -x ./exploit.py
```

## Recklinghausen Writeup

This program requires the flag as an argument. The flag has to be `CTFlearn{flag}`. I need to find the proper `flag` value.

```
└─[0] <> ./Recklinghausen
Welcome to the Recklinghausen Reversing Challenge!
Compile Options: ${CMAKE_CXX_FLAGS} -O0 -fno-stack-protector -mno-sse
Usage: Recklinghausen CTFlearn{flag}

└─[1] <> ./Recklinghausen CTFlearn{test}
Welcome to the Recklinghausen Reversing Challenge!
Compile Options: ${CMAKE_CXX_FLAGS} -O0 -fno-stack-protector -mno-sse
SORRY You did not find the flag :-( : CTFlearn{test}
```

### static analysis with r2
I start doing a static analysis with r2 to see a function graph.

```
└─[0] <> r2 Recklinghausen
[0x000012b0]> aaa
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze function calls (aac)
...

[0x000012b0]> afl
...
0x000010c0   14 494          main

[0x000012b0]> s main
[0x000010c0]> VV
```

![reckling_hausen_main](/media/ctfwritesup/reckling_hausen_main.png)

In the beginning, there is a time limit during checking whether the input has a flag or not.  The program exits if it takes more than 2 seconds during this input checking. It's probably for the case of using Debuggers.

```s
call sym get_wall_time()
...
lea rdi, obj.buffer
mov rbp, rax ; store the time into $rbp
...
call sym get_wall_time()
sub rax, rbp
cmp rax, 2
jg 0x1217 ; must be less than 2
```

After that, the program calls the `CheckMsg` function with the input flag. This function returns a boolean, which seems to check the flag value. So I visualized this function.
```s
mov rdi, r12
call sym CheckMsg(char const*)
test al, al
je 0x11e2
```

```
[0x00002390]> s sym.CheckMsg_char_const_
[0x00002390]> VV
```

#### Flag size
The `CheckMsg` checks the size of the flag compared with the size of `obj.msg5` variable at first.

```s
call sym.imp.strlen
movzx edx, byte [obj.msg5]
mov r8, rax
xor eax, eax
cmp rdx, r8
jne 0x23e5
```

#### Flag value validation 

After the size check, there is a loop.
The below part in the loop checks the flag value.

```s
movzx edx, byte [rbx + rax] ; copy char of the flag into edx
xor edx, esi
cmp byte [rdi + rax], dl ; byte [rdi + rax] == edx XOR esi
je 0x23d0
```
![rechking_check_msg](/media/ctfwritesup/reckling_check_msg.png)

It checks whether the result of `char of the flag XOR $rsi` is equal to the byte value of `$rdi+$rax*1`, which means that each char of the flag must be equal to the result of `$rsi XOR the byte value of $rdi+$rax` . This logic can be described as the statement below.
```
char of the flag ^ $rsi = [$rdi+$rax*1] => char of the flag = [$rdi+$rax*1] ^ $rsi
```

As the result of the static analysis, I got the following two things.
1. There is a size check of the flag
2. There is a loop to check the flag

### dynamic analysis with GDB

Based on the result of the static analysis, I want to check the actual values and get the flag.
1. The flag size
2. The actual flag that is checked during the loop

#### 1. The flag size
In order to check the flag dynamically, I set a breakpoint where it checks the flag size.
```
gdb-peda$ break *0x00005555555563a9
gdb-peda$ run CTFlearn{xxxx}
```

When the program comes to the breakpoint, the rdx register has a value, `0x21`, which is 33 as a digit. So the flag size is __33__.
```
gdb-peda$ i register rdx
rdx            0x21                0x21
```

#### 2. The actual flag that is checked during the loop
In order to check the actual value, I set a breakpoint where it checks the value of the flag, then executes the program with 33 characters.
```
gdb-peda$ break *0x5555555563e1
gdb-peda$ run CTFlearn{xxxxxxxxxxxxxxxxxxxxxxx}
```

When the program comes to the break point for the first time, I check whether the logic below is correct. 
```
char of the flag ^ $rsi = [$rdi+$rax*1] => char of the flag = [$rdi+$rax*1] ^ $rsi
```

The values of the corresponding registers are below.
```
gdb-peda$ i register $rsi
rsi            0x7e                0x7e
gdb-peda$ x/x $rdi+$rax
0x5555555590e2 <msg5+2>:        0x3d
```

With these values above, I calculate the first character of the flag, which has to be `C`.
The result of the calculation is 'C', so the logic seems to be correct. 
```shell
└─[0] <> printf '\x0x%X' $(( 0x7e ^ 0x3d )) | xxd -r -p
C
```
I also checked next some flags by continuing the debug and confirmed the flag starts with `CTFlearn{`.
Now it's time to exploit it with a script.

### exploit with gdb
Here is the script to get the flag. In order to continue the program until I gets the whole flag, I set the `ZF` flag manually.
```python
import gdb

gdb.execute('break *0x5555555563e1')
FLAG_SIZE = 33
dummy_flag = 'x' * FLAG_SIZE
gdb.execute('run {}'.format(dummy_flag))

flag = ''
for i in range(FLAG_SIZE):
    # get a correct char
    rsi = gdb.parse_and_eval('$rsi')
    data = gdb.execute('x/x $rdi+$rax*1', to_string=True)
    v = data.split(":")[1]
    # flag ^ $rsi = $v => flag = $rsi ^ $v
    flag += chr(rsi ^ int(v, 16))

    # set ZF flag
    eflags_raw = gdb.execute('i r $eflags', to_string=True)
    eflags = eflags_raw.split()[1]
    eflags = hex(int(eflags, 16) | (1 << 6)).strip("L") 
    gdb.execute("set $eflags = {}".format(eflags), to_string=True)

    gdb.execute('continue')

print(flag)
```

I get the flag by running the script with GDB.
```shell
gdb Recklinghausen -x ./exploit.py
```

## References
- [gdb.execute](https://programtalk.com/python-examples/gdb.execute/?ipage=3)