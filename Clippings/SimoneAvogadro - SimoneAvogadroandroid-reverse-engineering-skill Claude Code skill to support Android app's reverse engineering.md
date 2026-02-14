---
title: "SimoneAvogadro/android-reverse-engineering-skill: Claude Code skill to support Android app's reverse engineering"
source: "https://github.com/SimoneAvogadro/android-reverse-engineering-skill"
author:
  - "SimoneAvogadro"
published:
created: 2026-02-08
description: "Claude Code skill to support Android app's reverse engineering - SimoneAvogadro/android-reverse-engineering-skill"
tags:
  - "clippings/articles"
  - "_inbox"
---
# SimoneAvogadro/android-reverse-engineering-skill: Claude Code skill to support Android app's reverse engineering

![](https://opengraph.githubassets.com/e3d3e9e42069bbc0f56bfa061dfbb8d9a4b3aaba0736f570514d8522d49b08b8/SimoneAvogadro/android-reverse-engineering-skill)

> [!summary]+
> > This page details the GitHub repository `SimoneAvogadro/android-reverse-engineering-skill`, an Android Reverse Engineering & API Extraction Claude Code skill.
>
> It decompiles Android application files (APK, XAPK, JAR, AAR) to extract HTTP APIs, including Retrofit endpoints, OkHttp calls, hardcoded URLs, and authentication patterns.
>
> Key features include decompilation using tools like jadx and Fernflower/Vineflower, API extraction, call flow tracing, app structure analysis, and handling obfuscated code.
>
> Installation can be done via the Claude Code marketplace or a local clone. Usage is supported through a slash command (`/decompile`), natural language prompts, or direct execution of bash scripts.
>
> The repository requires Java JDK 17+ and `jadx`, with `Vineflower`/`Fernflower` and `dex2jar` recommended for enhanced functionality.
>
> It operates under an Apache 2.0 license and includes a disclaimer emphasizing lawful use.
>
> The repository has 63 stars, 12 forks, and 1 watcher, and is primarily written in Shell.

Claude Code skill to support Android app's reverse engineering

A Claude Code skill that decompiles Android APK/XAPK/JAR/AAR files and **extracts the HTTP APIs** used by the app — Retrofit endpoints, OkHttp calls, hardcoded URLs, authentication patterns — so you can document and reproduce them without the original source code.

## What it does

- **Decompiles** APK, XAPK, JAR, and AAR files using jadx and Fernflower/Vineflower (single engine or side-by-side comparison)
- **Extracts and documents APIs**: Retrofit endpoints, OkHttp calls, hardcoded URLs, auth headers and tokens
- **Traces call flows** from Activities/Fragments through ViewModels and repositories down to HTTP calls
- **Analyzes** app structure: manifest, packages, architecture patterns
- **Handles obfuscated code**: strategies for navigating ProGuard/R8 output

## Installation

### From GitHub (recommended)

Inside Claude Code, run:

```
/plugin marketplace add SimoneAvogadro/android-reverse-engineering-skill
/plugin install android-reverse-engineering@android-reverse-engineering-skill
```

The skill will be permanently available in all future sessions.

### From a local clone

git clone https://github.com/SimoneAvogadro/android-reverse-engineering-skill.git

Then in Claude Code:

```
/plugin marketplace add /path/to/android-reverse-engineering-skill
/plugin install android-reverse-engineering@android-reverse-engineering-skill
```

## Usage

### Slash command

```
/decompile path/to/app.apk
```

The scripts can also be used standalone:

```sh
# Check dependencies
bash plugins/android-reverse-engineering/skills/android-reverse-engineering/scripts/check-deps.sh

# Install a missing dependency (auto-detects OS and package manager)
bash plugins/android-reverse-engineering/skills/android-reverse-engineering/scripts/install-dep.sh jadx
bash plugins/android-reverse-engineering/skills/android-reverse-engineering/scripts/install-dep.sh vineflower

# Decompile APK with jadx (default)
bash plugins/android-reverse-engineering/skills/android-reverse-engineering/scripts/decompile.sh app.apk

# Decompile XAPK (auto-extracts and decompiles each APK inside)
bash plugins/android-reverse-engineering/skills/android-reverse-engineering/scripts/decompile.sh app-bundle.xapk

# Decompile with Fernflower
bash plugins/android-reverse-engineering/skills/android-reverse-engineering/scripts/decompile.sh --engine fernflower library.jar

# Run both engines and compare
bash plugins/android-reverse-engineering/skills/android-reverse-engineering/scripts/decompile.sh --engine both --deobf app.apk

# Find API calls
bash plugins/android-reverse-engineering/skills/android-reverse-engineering/scripts/find-api-calls.sh output/sources/
bash plugins/android-reverse-engineering/skills/android-reverse-engineering/scripts/find-api-calls.sh output/sources/ --retrofit
bash plugins/android-reverse-engineering/skills/android-reverse-engineering/scripts/find-api-calls.sh output/sources/ --urls
```
