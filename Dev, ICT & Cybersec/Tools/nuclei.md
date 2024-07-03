---
Description: A fast and customisable vulnerability scanner powered by simple YAML-based templates.
URL: https://docs.projectdiscovery.io/tools/nuclei/overview
---

# General working

Nuclei is a fast vulnerability scanner designed to probe modern applications, infrastructure, cloud platforms, and networks, aiding in the identification and mitigation of exploitable vulnerabilities.

At its core, Nuclei uses templates—expressed as straightforward YAML files, that delineate methods for detecting, ranking, and addressing specific security flaws.

## Scan a target using community rules

Default templates used by nuclei: from https://github.com/projectdiscovery/nuclei-templates

There are some exceptions regarding the templates that run by default:
- Certain tags and templates listed in the [default `.nuclei-ignore` file](https://github.com/projectdiscovery/nuclei-templates/blob/main/.nuclei-ignore) are not included.
- [Code Templates](https://docs.projectdiscovery.io/templates/protocols/code) require the `-code` flag to execute.
- [Headless Templates](https://docs.projectdiscovery.io/templates/protocols/headless) will not run unless you pass the `-headless` flag.
- [Fuzzing Templates](https://docs.projectdiscovery.io/template/protocols/http/fuzzing-overview) will not run unless you pass the `-fuzz` flag.
- A separate collection of [Fuzzing Templates](https://docs.projectdiscovery.io/templates/protocols/http/fuzzing-overview), located in a [different repository](https://github.com/projectdiscovery/fuzzing-templates), must be downloaded and configured separately for use.

```bash
$ nuclei -l target.txt 
```

## Scan a target using custom templates


```bash
$ nuclei -u https://0xbro.red -t templates1/ -t tempaltes2/
$ nuclei -u https://example.com -t https://cloud.projectdiscovery.io/public/tech-detect
```

## Scan a target using workflows

```bash
$ nuclei -u https://example.com -w workflows/
```


# Templating

>[!summary] RTFM
>Official documentation: https://docs.projectdiscovery.io/templates/introduction

Common components:
- matchers
- extractors
- variables
- flows
- interactsh

## Matchers

Matchers allow different type of flexible comparisons on protocol responses. [^matchers]

[^matchers]: [matchers](https://docs.projectdiscovery.io/templates/reference/matchers); docs.projectdiscovery.io

```yaml
    matchers-condition: and
    matchers:
      - type: word
        words:
          - "X-Powered-By: PHP"
          - "PHPSESSID"
        condition: or
        part: header

      - type: word
        words:
          - "PHP"
        part: body
```

## Extractors

Extractors can be used to extract and display in results a match from the response returned by a module. [^extractors]

[^extractors]: [extractors](https://docs.projectdiscovery.io/templates/reference/extractors); docs.projectdiscovery.io

```yaml
    extractors:
      - type: regex
        name: api
        part: body
        internal: true # Required for using dynamic variables
        regex:
          - "(?m)[0-9]{3,10}\\.[0-9]+"
```


## Variables

Variables can be used to declare some values which remain constant throughout the template. The value of the variable once calculated does not change. Variables can be either simple strings or DSL helper functions. If the variable is a helper function, it is enclosed in double-curly brackets `{{<expression>}}`. Variables are declared at template level. [^variables]

[^variables]: [variables](https://docs.projectdiscovery.io/templates/reference/variables); docs.projectdiscovery.io

```yaml
# Variable example using HTTP requests
id: variables-example

info:
  name: Variables Example
  author: pdteam
  severity: info

variables:
  a1: "value"
  a2: "{{base64('hello')}}"

http:
  - raw:
      - |
        GET / HTTP/1.1
        Host: {{FQDN}}
        Test: {{a1}}
        Another: {{a2}}
    stop-at-first-match: true
    matchers-condition: or
    matchers:
      - type: word
        words: 
          - "value"
          - "aGVsbG8="
```

## Flows

The template flow engine was introduced in nuclei v3, and brings two significant enhancements to Nuclei: [^flow]

- The ability to [conditionally execute requests](https://docs.projectdiscovery.io/templates/protocols/flow#conditional-execution)
- The [orchestration of request execution](https://docs.projectdiscovery.io/templates/protocols/flow#request-execution-orchestration)

[^flow]: [flow](https://docs.projectdiscovery.io/templates/protocols/flow); docs.projectdiscovery.io

```yaml
flow: |
  if(http(1)){
	http(2);
	log(template)
	if(template.links){
		for(i=0; i<template.links.length; i++){
			if(template.links[i].startWith(template.http_1_host)){
				set("link", template.links[i]);
				http(3);
			}
		}
	}
  }
```

## interactsh

Interactsh server runs multiple services and captures all the incoming requests (like burp collaborator). [^interactsh]
`{{interactsh-url}}` placeholder is supported in **http** and **network** requests. [^oob-testing]

[^interactsh]: [interactsh](https://docs.projectdiscovery.io/tools/interactsh/usage); docs.projectdiscovery.io
[^oob-testing]: [oob-testing](https://docs.projectdiscovery.io/templates/reference/oob-testing); docs.projectdiscovery.io

```yaml
  - raw:
      - |
        GET /plugins/servlet/oauth/users/icon-uri?consumerUri=https://{{interactsh-url}} HTTP/1.1
        Host: {{Hostname}}
```

# Video

<iframe width="660" height="415" src="https://www.youtube.com/embed/T83IR4Ba0io" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
