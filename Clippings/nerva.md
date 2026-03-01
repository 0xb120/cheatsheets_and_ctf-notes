---
title: "praetorian-inc/nerva: Fast service fingerprinting CLI for 120+ protocols (TCP/UDP/SCTP) - built by Praetorian"
source: https://github.com/praetorian-inc/nerva
author:
  - anushkavirgaonkar
published:
created: 2026-02-28
description: Fast service fingerprinting CLI for 120+ protocols (TCP/UDP/SCTP) - built by Praetorian - praetorian-inc/nerva
tags:
  - clippings/articles
  - tools
---
# praetorian-inc/nerva: Fast service fingerprinting CLI for 120+ protocols (TCP/UDP/SCTP) - built by Praetorian

![](https://opengraph.githubassets.com/2c31be1a9fb8744d45a8e80b2253db16ec5855f3372af4c3514fe875cf6738bc/praetorian-inc/nerva)

> [!summary]+
> > Nerva is a high-performance, open-source command-line interface (CLI) tool written in Go for service fingerprinting.
> It identifies over 120 network protocols across TCP, UDP, and SCTP transports, extracting rich metadata like versions and configurations.
> Key features include multi-transport support, detailed metadata extraction, a \"fast mode\" for quick scans, and flexible output formats (JSON, CSV, human-readable).
> Nerva can be installed via prebuilt binaries, `go install`, source, or Docker.
> It's designed to be pipeline-friendly, integrating with tools like Naabu, and is useful for penetration testing, asset discovery, CI/CD security, and bug bounty reconnaissance.
> The tool is a maintained fork of fingerprintx, originally developed by Praetorian's 2022 intern class.

Fast service fingerprinting CLI for 120+ protocols (TCP/UDP/SCTP) - built by Praetorian

> **High-performance service fingerprinting written in Go.** Identify 120+ network protocols across TCP, UDP, and SCTP transports with rich metadata extraction.

Nerva rapidly detects and identifies services running on open network ports. Use it alongside port scanners like [Naabu](https://github.com/projectdiscovery/naabu) to fingerprint discovered services, or integrate it into your security pipelines for automated reconnaissance.

## Installation

Download a prebuilt binary from the [Releases](https://github.com/praetorian-inc/nerva/releases) page.

Go:
```sh
go install github.com/praetorian-inc/nerva/cmd/nerva@latest
```

From source:
```sh
git clone https://github.com/praetorian-inc/nerva.git
cd nerva
go build ./cmd/nerva
./nerva -h
```

Docker:
```sh
git clone https://github.com/praetorian-inc/nerva.git
cd nerva
docker build -t nerva .
docker run --rm nerva -h
docker run --rm nerva -t example.com:80 --json
```

## Usage

```sh
nerva [flags]

TARGET SPECIFICATION:
  Requires host:port or ip:port format. Assumes ports are open.

EXAMPLES:
  nerva -t example.com:80
  nerva -t example.com:80,example.com:443
  nerva -l targets.txt
  nerva --json -t example.com:80
  cat targets.txt | nerva
```

### Examples

```sh
# Multiple targets
nerva -t example.com:22,example.com:80,example.com:443

# From file
nerva -l targets.txt --json -o results.json

# Fast mode (default ports only)
nerva -l large-target-list.txt --fast --json

# Asset Discovery Pipelines
naabu -host 10.0.0.0/24 -silent | nerva --json | jq '.protocol'
```
