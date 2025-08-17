---
title: CRTO Certificate
date: "2025-08-17T12:00:00.000Z"
template: "post"
draft: false
slug: "crto-certificate"
category: "Security"
tags:
  - "Certificate"
description: ""
socialImage: "/media/certificates/crto.png"
---

![CRTO certificate](/media/certificates/crto.png)

Last week, I took the CRTO exam offered by [Zero-Point Security](https://training.zeropointsecurity.co.uk/courses/red-team-ops).

## Journey
I began the course with three months of lab access in October 2024 and completed the material by December 2024. However, due to other priorities, I paused my studies for a while.
In June 2025, I restarted the course from the beginning and registered for an additional two weeks of lab time. During this period, I first practiced in the lab without Windows Defender enabled, and then repeated the exercises with Windows Defender active, which gave me much more confidence.

## The exam
The exam provides 4 days (48 hours of lab access) to complete the challenges. To pass, I needed to capture at least 6 out of 8 flags.
I was a bit stuck on the third flag, but after taking some breaks I managed to solve it. From there, progress was smoother, and after about 8 hours I successfully captured 6 flags and passed the exam. I decided not to attempt the 7th and 8th flags afterward.

In the next day, I received [the badge](https://eu.badgr.com/public/assertions/T341H1XDR9yPknbWFgJ3fA).

## What I learned
I really enjoyed the course and gained a lot of practical knowledge. This was my first time using Cobalt Strike, and I learned:

- Cobaltstrike
    - Basic concept and usage
    - Customization techniques
- Bypassing Windows Defender in some scenarios
- Post exploitation tools
    - Mimikatz
    - Rubeus
    - Powershell scripts such as PowerView

The lab also included Elasticsearch and Kibana, which I could leverage to collect and analyze data from an OPSEC perspective.

## Next steps
Since Zero-Point Security has launched [a new site](https://www.zeropointsecurity.co.uk/blog/new-site-launch), I plan to retake the CRTO exam there.
I’m also looking forward to attempting CRTL once it becomes available on the new platform.
