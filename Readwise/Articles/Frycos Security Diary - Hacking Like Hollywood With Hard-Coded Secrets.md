---
author: Frycos Security Diary
aliases: [Hacking Like Hollywood With Hard-Coded Secrets]
tags: [readwise/articles]
url: https://frycos.github.io/vulns4free/2023/11/07/hacking-like-hollywood.html
date: 2023-11-16
---
# Hacking Like Hollywood With Hard-Coded Secrets

![rw-book-cover](https://readwise-assets.s3.amazonaws.com/static/images/article2.74d541386bbf.png)

## Highlights

### id627220077

> “binwalk on steroids”, [unblob](https://unblob.org/), which even provides a Docker-ized version
> [View Highlight](https://read.readwise.io/read/01hfbxn2ehc2yf5h097h9pyc9r)
> #tools 

```bash
docker run \
  --rm \
  --pull always \
  -v ./unblob/output:/data/output \
  -v ./unblob/input:/data/input \
ghcr.io/onekey-sec/unblob:latest /data/input/$1
```

