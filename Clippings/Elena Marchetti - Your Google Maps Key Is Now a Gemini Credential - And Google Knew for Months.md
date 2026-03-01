---
title: Your Google Maps Key Is Now a Gemini Credential - And Google Knew for Months
source: https://awesomeagents.ai/news/google-api-keys-gemini-silent-privilege-escalation/
author:
  - Elena Marchetti
published: 2026-02-27
created: 2026-02-28
description: Truffle Security found 2,863 public Google API keys that silently gained access to Gemini AI endpoints, exposing private data and racking up charges with no warning to developers.
tags:
  - clippings/articles
---
# Your Google Maps Key Is Now a Gemini Credential - And Google Knew for Months

![](https://awesomeagents.ai/images/news/google-api-keys-gemini-silent-privilege-escalation_hu_5dd209dff249b343.jpg)

> [!summary]+
> Google API keys, previously considered non-sensitive and often publicly exposed as per Google's own documentation for services like Maps and Firebase, have silently become credentials for Gemini AI endpoints.
   Security researchers at Truffle Security discovered 2,863 such keys belonging to major financial institutions, security companies, and even Google itself, that now authenticate to Gemini.
   This \"silent privilege escalation\" occurs because Google API keys are project-scoped; when the Generative Language API (Gemini) is enabled in a project, all existing keys in that project gain access without notification.
   Attackers with these exposed keys can access uploaded files, read cached content, run up charges, and potentially reach connected services like Drive or Calendar.
   Google initially dismissed the issue as \"intended behavior\" but reclassified it as a \"Bug\" after Truffle provided evidence, including Google's own exposed keys.
   While Google has implemented proactive measures to block leaked keys from accessing Gemini and will default new AI Studio keys to Gemini-only scope, it has not confirmed notifying all 2,863 affected project owners individually.
   The article recommends auditing Google Cloud projects for the Generative Language API, rotating publicly exposed keys, applying API key restrictions, monitoring GCP billing, and reviewing third-party app permissions.

### How the escalation works

The mechanism is deceptively simple. Google API keys (formatted as `AIza...` strings) are project-scoped, not service-scoped. When anyone on a Google Cloud project enables the Generative Language API - the service that powers Gemini - every existing API key in that project silently inherits access to Gemini endpoints. No new key is issued. No permission prompt appears. No email is sent.

An attacker with one of those public keys can hit endpoints like:

```
GET https://generativelanguage.googleapis.com/v1beta/files?key=AIza...
GET https://generativelanguage.googleapis.com/v1beta/cachedContents?key=AIza...
```

These endpoints expose uploaded datasets, documents, cached context, and anything else the project owner stored through the Gemini API.