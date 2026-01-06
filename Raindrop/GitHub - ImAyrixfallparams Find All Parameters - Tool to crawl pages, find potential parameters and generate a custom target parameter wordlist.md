---
raindrop_id: 1355170712
raindrop_highlights:
  68d298f852f45e8876c079c9: b952100e538bb9799595a33748fae2f8
  68d298fd0e39366081da346f: 66c850796b80993114e8d698173f4e27
  68d2990808ae98bc80ddd033: 927451c2b3f241dade783c3d5be9c7ef
  68d299116ce16004f427219c: b11745fbd451b827a23acd6538a70fef
  68d2991c0d9bc9bcb7b6e297: dc13c567af5637dee0cfdd7b7c4fb4a7
title: "GitHub - ImAyrix/fallparams: Find All Parameters - Tool to crawl pages, find potential parameters and generate a custom target parameter wordlist"
description: Find All Parameters - Tool to crawl pages, find potential parameters and generate a custom target parameter wordlist
source: https://github.com/ImAyrix/fallparams
created: 1758614344422
type: link
tags:
  - Tools
aliases:
  - fallparams
---
# GitHub - ImAyrix/fallparams: Find All Parameters - Tool to crawl pages, find potential parameters and generate a custom target parameter wordlist

![](https://opengraph.githubassets.com/988d19296724e739f243310e41f23d33a370d680b227cfc7c6b7aca367ac2f32/ImAyrix/fallparams)

> [!summary]
> Find All Parameters - Tool to crawl pages, find potential parameters and generate a custom target parameter wordlist


Find All Parameters - Tool to crawl pages, find potential parameters and generate a custom target parameter wordlist

## Installation
```sh
go install github.com/ImAyrix/fallparams@latest
```

## Usage
### Start

To create a parameter wordlist suitable for the page you are working on, simply provide the page link to fallparams.

`fallparams -u "https://target.tld/page"`

If you have many URLs for which you want to create a parameter wordlist, save all your URLs in a file and then provide the file name to fallparams.

`fallparams -u "/path/to/file.txt"`

You can also save the entire HTTP packet to a file and use it as an input for the tool, ensuring that it sends the exact same headers, cookies, and other details in the requests.

`fallparams -r "request.txt"`

### More Parameters

One of the great features of fallparams is its ability to use [katana](../Dev,%20ICT%20&%20Cybersec/Tools/katana.md) to crawl the links provided as input, resulting in more links and eventually creating a comprehensive parameter wordlist.

`fallparams -u "https://target.tld/page" -crawl`

### Output

To enhance the quality of your parameter wordlist, it's crucial to filter out noisy or irrelevant words that may enter during creation. One effective way to achieve this is by setting a specific character limit for each parameter.

`fallparams -u "https://target.tld/page" -max-length 30`

You can also specify the minimum number of characters for each parameter.

`fallparams -u "https://target.tld/page" -min-length 3`

By default, the generated parameter wordlist is saved in the `parameters.txt` file. However, you can customize the name and path of the output file as needed.

`fallparams -u "https://target.tld/page" -output "custom_name.txt"`