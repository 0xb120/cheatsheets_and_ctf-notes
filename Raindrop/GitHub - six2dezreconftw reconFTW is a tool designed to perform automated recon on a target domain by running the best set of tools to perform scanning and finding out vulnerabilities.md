---
raindrop_id: 1474469343
raindrop_highlights:
  6932bcc0a8ccb40ed45e8b39: 0a4c67e5f4e44b3be0c69373a42523bb
  6932bcd695af055df9a7432a: 77ce935226c7dd13fac2689275b8fdbc
  6932bcf36df3deec9c8b71af: e99ac2b11712470d051eb486a7c7b492
  6932bd071ff4a506fb15c361: 68ddd761f88a1090c0468047ed88324c
  6932bd270d8ad22e052b0010: b12b3ee4abfa66b6850c2eb01e08a482
  6932bd6995af055df9a76989: abd78fa8fbec8a6113fa6d3f9fb018d1
title: "GitHub - six2dez/reconftw: reconFTW is a tool designed to perform automated recon on a target domain by running the best set of tools to perform scanning and finding out vulnerabilities"

description: |-
  reconFTW is a tool designed to perform automated recon on a target domain by running the best set of tools to perform scanning and finding out vulnerabilities - six2dez/reconftw

source: https://github.com/six2dez/reconftw

created: 1764926374591
type: link
tags:
  - "_index"

 
  - "Tools"

---
# GitHub - six2dez/reconftw: reconFTW is a tool designed to perform automated recon on a target domain by running the best set of tools to perform scanning and finding out vulnerabilities

![](https://repository-images.githubusercontent.com/325671936/e190df94-4fc5-4335-9b73-23e04e060768)

> [!summary]
> reconFTW is a tool designed to perform automated recon on a target domain by running the best set of tools to perform scanning and finding out vulnerabilities - six2dez/reconftw





reconFTW is a tool designed to perform automated recon on a target domain by running the best set of tools to perform scanning and finding out vulnerabilities
reconFTW is a powerful automated reconnaissance tool designed for security researchers and penetration testers. It streamlines the process of gathering intelligence on a target by performing subdomain enumeration, vulnerability scanning, OSINT and more. With a modular design, extensive configuration options, and support for distributed scanning via AX Framework, reconFTW is built to deliver comprehensive results efficiently.

reconFTW leverages a wide range of techniques, including passive and active subdomain discovery, web vulnerability checks (e.g., XSS, SSRF, SQLi), OSINT, directory fuzzing, port scanning and screenshotting. It integrates with cutting-edge tools and APIs to maximize coverage and accuracy, ensuring you stay ahead in your reconnaissance efforts.
💿 Installation
Quickstart
Clone and install
git clone https://github.com/six2dez/reconftw
cd reconftw
./install.sh --verbose
Run a scan (full + resume)
./reconftw.sh -d example.com -r
Minimal run (passive-only footprint)
./reconftw.sh -d example.com -p

Tip: re-run ./install.sh --tools later to refresh the toolchain without reinstalling system packages.
Docker

Pull the Image:

docker pull six2dez/reconftw:main

Run the Container:

docker run -it --rm \
  -v "${PWD}/OutputFolder/:/reconftw/Recon/" \
  six2dez/reconftw:main -d example.com -r

For a list of targets, bind the list file into the container and reference the in-container path:

docker run -it --rm \
  -v "${PWD}/domains.txt:/reconftw/domains.txt:ro" \
  -v "${PWD}/OutputFolder/:/reconftw/Recon/" \
  six2dez/reconftw:main -l /reconftw/domains.txt -r

View Results:

Results are saved in the OutputFolder directory on the host (not inside the container).
💻 Faraday Support

reconFTW integrates with Faraday for web-based reporting and vulnerability management.

Setup: Install Faraday, authenticate via faraday-cli, and configure the workspace in reconftw.cfg (FARADAY_SERVER, FARADAY_USER, FARADAY_PASS, FARADAY_WORKSPACE).
Usage: Enable with FARADAY=true in reconftw.cfg.