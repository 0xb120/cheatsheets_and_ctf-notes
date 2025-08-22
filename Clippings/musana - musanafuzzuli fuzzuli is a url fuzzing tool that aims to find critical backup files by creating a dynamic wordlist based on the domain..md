---
title: "musana/fuzzuli: fuzzuli is a url fuzzing tool that aims to find critical backup files by creating a dynamic wordlist based on the domain."
source: "https://github.com/musana/fuzzuli"
author:
  - "musana"
published:
created: 2025-08-19
description: "fuzzuli is a url fuzzing tool that aims to find critical backup files by creating a dynamic wordlist based on the domain. - musana/fuzzuli"
tags:
  - "clippings/articles"
  - "_inbox"
---
# musana/fuzzuli: fuzzuli is a url fuzzing tool that aims to find critical backup files by creating a dynamic wordlist based on the domain.

![](https://opengraph.githubassets.com/7b7ebc36c576351ca2bb8c8eea7ab350b7c0e53bf0525026be5efd27da948fe6/musana/fuzzuli)

> [!summary]
> 

fuzzuli is a url fuzzing tool that aims to find critical backup files by creating a dynamic wordlist based on the domain.

**fuzzuli** is a url fuzzing tool that aims to find critical backup files by creating a dynamic wordlist based on the domain.

**fuzzuli** consists of two main parts. One of them creates a wordlist dynamically. The other sends the HTTP request and checks the response for backup/sensitive file.

Flow chart of **fuzzuli** is the following.

# Installation Instructions

go install -v github.com/musana/fuzzuli@latest