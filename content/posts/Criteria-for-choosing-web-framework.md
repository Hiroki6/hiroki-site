---
title: Criteria for choosing web framework (Scala)
date: "2023-01-21T12:00:00.000Z"
template: "post"
draft: false
slug: "criteria-for-web-framework"
category: "Scala"
tags:
  - "Scala"
  - "Web"
description: ""
socialImage: "/media/hacker.png"
---

I picked up http4s and Scalatra as example frameworks. This article is not about comparison of Scala http frameworks.
I will write down these criterias below. These criterias might not cover all topics.
- [Project overview](#project-overview)
- [DSL & Syntax](#dsl--syntax)
- [Middleware](#middleware)
- [Testing](#testing)
- [JSON handling](#json-handling)
- [Swagger integration](#swagger-integration)
- [Others](#others)

### Project overview
- Compatibility
If the library isn't compatible with libraries or versions that you use, it would be difficult to use it.

- Project activity
If the library is not active, you might have a problem later.
For example, if the library doesn't support Scala `2.13`, you can't use Scala `2.13` in your project.

- Vulnerability
It's related to project activity. If the library leaves a critical vulnerability, it might lead to a security issue.
Otherwise, you have to apply to a patch by yourself.

### Performance
- blocking or not (servlet, etc)
- streaming

### DSL & Syntax

### Middleware

### Testing

### JSON handling

### Swagger integration

### Others
- http2 support
- cors
- graceful shutdown

- thread pools

- dependency injection (?)

- effects (?)
- prometheus
- templating

## Conclusion