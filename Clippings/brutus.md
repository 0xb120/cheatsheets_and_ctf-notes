---
title: "praetorian-inc/brutus: Fast, zero-dependency credential testing tool in Go. Brute force SSH, MySQL, PostgreSQL, Redis, MongoDB, SMB, and 20+ protocols. Hydra alternative with native fingerprintx/naabu pipeline integration."
source: https://github.com/praetorian-inc/brutus
author:
  - noah-tutt-praetorian
published:
created: 2026-02-28
description: Fast, zero-dependency credential testing tool in Go. Brute force SSH, MySQL, PostgreSQL, Redis, MongoDB, SMB, and 20+ protocols. Hydra alternative with native fingerprintx/naabu pipeline integration. - praetorian-inc/brutus
tags:
  - clippings/articles
  - tools
---
# praetorian-inc/brutus: Fast, zero-dependency credential testing tool in Go. Brute force SSH, MySQL, PostgreSQL, Redis, MongoDB, SMB, and 20+ protocols. Hydra alternative with native fingerprintx/naabu pipeline integration.

![](https://opengraph.githubassets.com/140f8c3b7cc8e502970fc95cc431680a6bc2fa23353b989fb2ae41a98e0b9d25/praetorian-inc/brutus)

> [!summary]+
> > Brutus is a modern, multi-protocol credential testing tool built in pure Go.
> It functions as a single, zero-dependency binary for cross-platform use (Linux, macOS, Windows).
> Designed as an alternative to tools like [hydra](../Dev,%20ICT%20&%20Cybersec/Tools/hydra.md), Brutus supports 24 protocols (e.g., SSH, RDP, MySQL, SMB, HTTP Basic Auth) and integrates seamlessly with reconnaissance pipelines like `fingerprintx` and `naabu`.
> Key features include embedded SSH bad keys, JSON streaming output, a Go library for automation, and robust error handling.
> Use cases span penetration testing, red team operations, private key spraying, and security validation.
> It offers experimental AI-powered credential detection for HTTP services using Claude Vision and Perplexity, and advanced RDP features including sticky keys backdoor detection and exploitation (command execution, interactive web terminal) via a Rust/WASM implementation.
> The project has 120 stars, 11 forks, 5 releases, and is actively developed by 7 contributors.

Fast, zero-dependency credential testing tool in Go. Brute force SSH, MySQL, PostgreSQL, Redis, MongoDB, SMB, and 20+ protocols. Hydra alternative with native fingerprintx/naabu pipeline integration.

## Overview

Brutus is a multi-protocol authentication testing tool designed to address a critical gap in offensive security tooling: [Password Attacks](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Password%20Attacks.md) and efficient credential validation across diverse network services.

Built in Go as a single binary with zero external dependencies, Brutus integrates seamlessly with [fingerprintx](https://github.com/praetorian-inc/fingerprintx) for automated service discovery, enabling operators to rapidly identify and test authentication vectors across entire network ranges.

**Key features:**

- **Zero dependencies:** Single binary, cross-platform (Linux, Windows, macOS)
- **24 protocols:** SSH, RDP, MySQL, PostgreSQL, MSSQL, Redis, SMB, LDAP, WinRM, SNMP, HTTP Basic Auth, and more
- **Pipeline integration:** Native support for fingerprintx and naabu workflows
- **Embedded bad keys:** Built-in collection of known SSH keys (Vagrant, F5, ExaGrid, etc.)
- **Go library:** Import directly into your security automation tools
- **Production ready:** Rate limiting, connection pooling, and comprehensive error handling

## Use Cases

### Private Key Spraying

Found a private key on a compromised system? Spray it across the network to find where else it grants access:

```sh
# Discover SSH services and spray a found private key
naabu -host 10.0.0.0/24 -p 22 -silent | \
  fingerprintx --json | \
  brutus -u root,admin,ubuntu,deploy -k /path/to/found_key --json
```

This pipeline discovers all SSH services, identifies them with fingerprintx, and tests the compromised key against common usernames—revealing lateral movement opportunities in seconds.

### Web Admin Panel Testing

Discover HTTP services with Basic Auth and test default credentials:

```sh
# Discover and test admin panels across a network
naabu -host 10.0.0.0/24 -p 80,443,3000,8080,9090 -silent | \\
  fingerprintx --json | \\
  brutus --json
```

## Installation
```sh
# Linux (amd64)
curl -L https://github.com/praetorian-inc/brutus/releases/latest/download/brutus-linux-amd64.tar.gz | tar xz
sudo mv brutus /usr/local/bin/

# macOS (Apple Silicon)
curl -L https://github.com/praetorian-inc/brutus/releases/latest/download/brutus-darwin-arm64.tar.gz | tar xz
sudo mv brutus /usr/local/bin/

# macOS (Intel)
curl -L https://github.com/praetorian-inc/brutus/releases/latest/download/brutus-darwin-amd64.tar.gz | tar xz
sudo mv brutus /usr/local/bin/
```

```powershell
# Windows (PowerShell)
Invoke-WebRequest \-Uri https://github.com/praetorian\-inc/brutus/releases/latest/download/brutus\-windows\-amd64.zip \-OutFile brutus.zip
Expand-Archive \-Path brutus.zip \-DestinationPath .
Remove-Item brutus.zip
```

```sh
go install github.com/praetorian-inc/brutus/cmd/brutus@latest
```

### Basic Usage

```sh
# Test SSH with embedded badkeys (tested by default)
brutus --target 192.168.1.100:22 --protocol ssh

# Test with specific credentials
brutus --target 192.168.1.100:22 --protocol ssh -u root -p toor

# Test with username and password lists
brutus --target 192.168.1.100:22 --protocol ssh -U users.txt -P passwords.txt

# Test MySQL database
brutus --target 192.168.1.100:3306 --protocol mysql -u root -p password

# Test SSH with a specific private key
brutus --target 192.168.1.100:22 --protocol ssh -u deploy -k /path/to/id_rsa

# Increase threads for faster testing
brutus --target 192.168.1.100:22 --protocol ssh -t 20

# JSON output for scripting
brutus --target 192.168.1.100:22 --protocol ssh --json
```