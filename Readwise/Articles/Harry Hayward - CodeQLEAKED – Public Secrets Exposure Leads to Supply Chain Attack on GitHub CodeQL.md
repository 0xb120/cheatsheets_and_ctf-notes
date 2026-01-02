---
author: Harry Hayward
aliases:
  - CodeQLEAKED – Public Secrets Exposure Leads to Supply Chain Attack on GitHub CodeQL
tags:
  - readwise/articles
url: ?__readwiseLocation=
date: 2025-04-22
---
# CodeQLEAKED – Public Secrets Exposure Leads to Supply Chain Attack on GitHub CodeQL

See [GitHub Actions](../../Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/GitHub%20Actions.md)

## What are Workflow Artifacts?

We found the publicly exposed secret in a [GitHub Actions workflow artifact](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/storing-and-sharing-data-from-a-workflow).
GitHub Actions workflows can upload workflow “artifacts” to GitHub Actions. Workflow artifacts can be any file and are saved by that workflow for later use. By default, artifacts are publicly accessible to anyone with read access to the repository and are stored for up to 90 days. [](https://read.readwise.io/read/01jsef5r79428nrzs66qde3w8j)

## Investigating Impact of exposed GITHUB_TOKEN

Secrets compromise is cool, but what can we do with this token? The impact of a compromised *GITHUB_TOKEN* is minimal if it only has read permissions.
The easiest way to determine the privileges of a *GITHUB_TOKEN* is to look at workflow logs. To investigate this, I navigated to the “Setup Job” step of the workflow that uploaded the token. [](https://read.readwise.io/read/01jsef8m92mtdzbcaf0pmyc85j)

We could spend a lot of time talking about [each privilege](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/controlling-permissions-for-github_token), but let’s focus on the ones that are particularly interesting.

- Contents: write – Allows the token to create branches, create tags, and upload release artifacts. 
- Actions: write – Allows you to work with Actions, including trigger workflow_dispatch events. 
- Packages: write – Allows the token to upload packages. [](https://read.readwise.io/read/01jsef95y55xmzqdagf4xkesyk)

These tokens are **only valid for the duration of their specific workflow job**. That means that once the job is over, the token is useless. Three things needed to happen for an attacker to be able to abuse this token:
1. The token needs to have some sort of write privileges (already confirmed).
2. The token needs to use V4 of the upload artifact API, as that is the only version that allows you to retrieve an artifact before the job is complete (and after the job is complete, the token is invalid.)
3. The time between uploading the artifact and completing the job needs to be great enough for us to download, extract, and use the token. [](https://read.readwise.io/read/01jsefacp1qer17h18ajdg9h3d)

To test this, I made a Python script *artifact_racer.py*. Artifact racer performs the following actions.
1. Continuously queries the *github/codeql-action* GitHub repository until it sees a “PR Check – Debug artifacts after failure” workflow begin.
2. Monitors the running workflow for artifacts.
3. Once it sees a “PR Check – Debug artifacts after failure” workflow run, it downloads the artifact and extracts the *GITHUB_TOKEN*.
- Shelling out for file operations and downloads was key to increasing the speed, although there are probably ways to make it even faster.
1. Uses the *GITHUB_TOKEN* to make a new branch.
2. Use the *GITHUB_TOKEN* to push an empty file named `poc.txt` to that branch.
3. **Makes a new tag** for that commit. [](https://read.readwise.io/read/01jsefdcw9f7x7166ftbtkqj4k)