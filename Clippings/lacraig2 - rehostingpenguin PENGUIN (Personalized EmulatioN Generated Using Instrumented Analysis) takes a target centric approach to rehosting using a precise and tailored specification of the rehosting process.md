---
title: "rehosting/penguin: PENGUIN (Personalized EmulatioN Generated Using Instrumented Analysis) takes a target centric approach to rehosting using a precise and tailored specification of the rehosting process"
source: "https://github.com/rehosting/penguin"
author:
  - "lacraig2"
published:
created: 2025-07-29
description: "PENGUIN (Personalized EmulatioN Generated Using Instrumented Analysis) takes a target centric approach to rehosting using a precise and tailored specification of the rehosting process - rehosting/penguin"
tags:
  - "clippings/articles"
  - "_inbox"
  - "tools"
---
# rehosting/penguin: PENGUIN (Personalized EmulatioN Generated Using Instrumented Analysis) takes a target centric approach to rehosting using a precise and tailored specification of the rehosting process

![](https://opengraph.githubassets.com/05787a53ff53cb078c5485a5dc65d2c2c3a5e9c602037e8bcda3e8c528925094/rehosting/penguin)

> [!summary]
> The page describes **Penguin (Personalized EmulatioN Generated Using Instrumented Analysis)**, a firmware rehosting tool that uses a target-centric approach with precise, tailored configuration files. Its primary goal is to get firmware running in a virtual environment for interaction and behavior monitoring.

The typical workflow involves four phases:
1.  **Obtain target filesystem:** Using `fw2tar` to get a tarball of the firmware root filesystem.
2.  **Generate initial configuration:** Creating a project directory and a `config.yaml` file via static analysis using `penguin init`.
3.  **Run rehosting:** Executing the rehosting process as defined in the configuration using `penguin run`.
4.  **Refine rehosting configuration:** Interacting with the running system via a Penguin root shell (telnet) or Penguin VPN, examining dynamic analysis outputs (like `console.log`), and iteratively updating the configuration.

Installation involves pulling the `rehosting/penguin` Docker container and running an install script (system-wide or local). The project emphasizes its detailed documentation available in the `docs` directory, covering typical workflows, playbooks, plugin details, and configuration file schemas. The project is open-source, hosted on GitHub, and has 11 stars, 0 forks, and 9 watchers, with Python as its primary language.

[penguin](https://github.com/rehosting/penguin)

PENGUIN (Personalized EmulatioN Generated Using Instrumented Analysis) takes a target centric approach to rehosting using a precise and tailored specification of the rehosting process

# Penguin: Configuration Based Rehosting

Penguin is a firmware rehosting tool designed to get your firmware up and running in a virtual environment where you can interact with it and closely monitor its behavior. Unlike other rehosting tools, Penguin takes a **target-centric** approach to rehosting where the specific details of each rehosting process are stored in a configuration file that you can examine and edit as necessary.

There are typically four phases to rehosting a system with Penguin:

### Obtain target filesystem

Before you start with Penguin, you'll need an archive of a firmware root filesystem. This is a tarball of the root filesystem with permissions and ownership preserved. You can generate this with the [fw2tar](https://github.com/rehosting/fw2tar) utility or by hand. [Installing Penguin](https://github.com/rehosting/#installation) will also install fw2tar without requiring an additional container. Updating from an old penguin version may require rerunning the installation script to install fw2tar.

```
fw2tar your_fw.bin
```

### Generate initial configuration

Once you have a root filesystem, you can generate an initial rehosting configuration based on a static analysis of the filesystem. This initial configuration is stored within a "project directory" which will hold the config, static analysis results, and the output from every dynamic analysis you run.

To generate your initial configuration you'll use the `init` subcommand to `penguin`. This will generate a configuration for the provided firmware root filesystem. By default the configuration will be stored in `./projects/<firmware_name>/config.yaml`. You can specify a different output directory with the `--output` flag.

`penguin init your_fw.rootfs.tar.gz --output projects/your_fw `

### Run rehosting specified by configuration

To run a configuration, use the `run` subcommand. This will run the rehosting as specified in the configuration file and report dynamic analysis results in a "results directory." By default this directory will be within the project directory at `<project directory>/results/<auto-incrementing number>`. You can also specify an output directory with the `--output` flag and replace an existing directory with `--force`.

`penguin run projects/your_fw/config.yaml --output projects/your_fw/results/0 `

### Refine rehosting configuration

### Examples

You may find examples of past rehostings useful for learning how to use penguin. Examples are available at [https://github.com/rehosting/examples](https://github.com/rehosting/examples)

## Pull container and install `penguin`

docker pull rehosting/penguin

docker run rehosting/penguin penguin\_install | sudo sh

docker run rehosting/penguin penguin\_install.local | sh