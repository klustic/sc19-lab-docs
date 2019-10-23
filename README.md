# I Can't Be Hacked, I'm Serverless

## Introduction

With server hardware, operating system, and interpreter managed by a cloud provider, the devops engineer's primary security concern is application security.

The OWASP Top Ten list has long been a staple when understanding trends in web application security risks. OWASP has built on that program by translating those risks to the serverless environment. These labs demonstrate exploitation of many of the OWASP Top Ten 2017 - Serverless vulnerabilities.

## Setup

You should be able to do most of this work with your browser and a text editor.

The later labs will require Python (2.7) to be installed:
- [Windows](https://www.python.org/ftp/python/2.7.17/python-2.7.17.amd64.msi)
- [Mac](https://www.python.org/ftp/python/2.7.17/python-2.7.17-macosx10.9.pkg)
- Linux - use your package manager

For cryptographic work (e.g. hashing) and other data translations, you can probably get by using [CyberChef](https://gchq.github.io/CyberChef).

## Premise

These labs traverse a series of vulnerabilities in a serverless To Do List, which allows you to
- Create different lists for your tasks
- Export/Import your tasks
- Mark tasks "complete" and remove them from the list entirely

You will learn the vulnerabilities as you go, and eventually capture a flag from the environment as you achieve remote code execution on the application's container!

This lab is implemented in AWS Lambda.

## Labs

- [Lab 1](./Lab1.md): Application familiarity
- [Lab 2](./Lab2.md): File disclosure
- [Lab 3](./Lab3.md): Code execution
- [Lab 4](./Lab4.md): Serverless injection

## Vulnerabilities covered

| Designation | Description | Covered in |
| :---: | --- | --- |
| A1:2017 | Injection | [Lab 4](./Lab4.md) |
| A2:2017 | Broken Authentication | [Lab 3](./Lab3.md) |
| A3:2017 | Sensitive Data Exposure | [Lab 2](./Lab2.md) |
| A4:2017 | XML External Entities (XXE) | [Lab 2](./Lab2.md) |
| A5:2017 | Broken Access Control | [Lab 3](./Lab3.md) (extra credit) |
| A6:2017 | Security Misconfiguration | [Lab 2](./Lab2.md) |
| A7:2017 | Cross-Site Scripting (XSS) | [Lab 1](./Lab1.md) |
| A8:2017 | Insecure Deserialization | [Lab 3](./Lab3.md) |
| A9:2017 | Using Components with Known Vulnerabilities | [Lab2](./Lab2.md) |
| A10:2017 | Insufficient Logging/Monitoring | [Lab4](./Lab4.md) |
