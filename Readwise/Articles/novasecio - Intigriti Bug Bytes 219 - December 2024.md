---
author: novasecio
aliases: []
tags: [readwise/articles]
url: https://blog.intigriti.com/bug-bytes/bug-bytes-219-december-2024
date: 2024-12-19
---
# Intigriti Bug Bytes #219 - December 2024 🎅

![rw-book-cover](https://blog.intigriti.com/icon.svg)

## Highlights

[You’re missing out on critical misconfigurations if you aren’t using Misconfig Mapper!](https://github.com/intigriti/misconfig-mapper) A simple tool that can help you automate security misconfiguration detection on your list of targets! ^4a91a1
> [View Highlight](https://read.readwise.io/read/01jfdbmckt2683hwk6pg21cvbf)
> #tools 


Misconfig Mapper is a project by Intigriti for the community to help you find, detect and resolve common security misconfigurations in various popular services, technologies and SaaS-based solutions that your targets use!

Documentation: https://bugology.intigriti.io/misconfig-mapper-docs/readme/introduction

## Install

```sh
go install -v github.com/intigriti/misconfig-mapper/cmd/misconfig-mapper@latest
```

### Usage

```sh
$ ./misconfig-mapper -list-services
$ ./misconfig-mapper -target "yourcompanyname" -service 1 -delay 1000
$ ./misconfig-mapper -target "yourcompanyname" -service 1 -skip-misconfiguration-checks true
$ ./misconfig-mapper -target "yourcompanyname" -service "drupal"
```