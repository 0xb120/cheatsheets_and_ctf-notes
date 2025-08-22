---
title: "How GitHub uses CodeQL to secure GitHub"
source: "https://github.blog/engineering/how-github-uses-codeql-to-secure-github/?ref=blog.exploits.club&__readwiseLocation="
author:
  - "Brandon Stewart"
published: 2025-02-12
created: 2025-08-13
description: "How GitHub’s Product Security Engineering team manages our CodeQL implementation at scale and how you can, too."
tags:
  - "clippings/articles"
  - "_inbox"
---
# How GitHub uses CodeQL to secure GitHub

![](https://github.blog/wp-content/uploads/2024/02/Security-DarkMode-2-2.png?fit=1200%2C630)

> [!summary]
> GitHub's Product Security Engineering team secures its code at scale using CodeQL, leveraging default and advanced setups with custom query packs published to GitHub Container Registry for efficient deployment, alongside multi-repository variant analysis to find vulnerabilities and enforce security controls, with resources available for custom query development.

How GitHub’s Product Security Engineering team manages our CodeQL implementation at scale and how you can, too.

We use GitHub Advanced Security (GHAS) to discover, track, and remediate vulnerabilities and enforce secure coding standards at scale. One tool we rely heavily on to analyze our code at scale is [CodeQL](https://codeql.github.com/).

The following post will detail how we use CodeQL to keep GitHub secure and how you can apply these lessons to your own organization. You will learn why and how we use:

- Custom query packs (and how we create and manage them).
- Custom queries.
- Variant analysis to uncover potentially insecure programming practices.

## [Enabling CodeQL at scale](https://github.blog/engineering/how-github-uses-codeql-to-secure-github/?ref=blog.exploits.club&__readwiseLocation=#enabling-codeql-at-scale)

We employ CodeQL in a variety of ways at GitHub.

1. **[Default setup](https://docs.github.com/en/code-security/code-scanning/enabling-code-scanning/configuring-default-setup-for-code-scanning)** with the [default and security-extended query suites](https://docs.github.com/en/code-security/code-scanning/managing-your-code-scanning-configuration/codeql-query-suites#built-in-codeql-query-suites)  
	Default setup with the default and security-extended query suites meets the needs of the vast majority of our over 10,000 repositories. With these settings, pull requests automatically get a security review from CodeQL.
2. **[Advanced setup](https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/configuring-advanced-setup-for-code-scanning) with a custom query pack**  
	A few repositories, like our large Ruby monolith, need extra special attention, so we use advanced setup with a [query pack](https://docs.github.com/en/code-security/codeql-cli/getting-started-with-the-codeql-cli/customizing-analysis-with-codeql-packs#about-codeql-packs) containing custom queries to really tailor to our needs.
3. **[Multi-repository variant analysis](https://docs.github.com/en/code-security/codeql-for-vs-code/getting-started-with-codeql-for-vs-code/running-codeql-queries-at-scale-with-multi-repository-variant-analysis) (MRVA)**  
	To conduct variant analysis and quick auditing, we use MRVA. We also write custom CodeQL queries to detect code patterns that are either specific to GitHub’s codebases or patterns we want a security engineer to manually review.

## [Publishing our CodeQL query pack](https://github.blog/engineering/how-github-uses-codeql-to-secure-github/?ref=blog.exploits.club&__readwiseLocation=#publishing-our-codeql-query-pack)

query pack to [GitHub Container Registry (GCR)](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)

So while it’s *possible* to deploy custom CodeQL query files directly to a repository, we recommend publishing CodeQL queries as a query pack to the GCR for easier deployment and faster iteration.

## [Creating our query pack](https://github.blog/engineering/how-github-uses-codeql-to-secure-github/?ref=blog.exploits.club&__readwiseLocation=#creating-our-query-pack)

When setting up our custom [query pack](https://docs.github.com/en/code-security/codeql-cli/using-the-advanced-functionality-of-the-codeql-cli/creating-and-working-with-codeql-packs#codeql-pack-structure), we faced several considerations

To ensure our custom queries remain maintainable and concise, we extend classes from the [default query suite](https://docs.github.com/en/code-security/code-scanning/managing-your-code-scanning-configuration/ruby-built-in-queries), such as the `ruby-all` library.

We develop our queries against the latest version of the `ruby-all` package, ensuring we’re always working with the most up-to-date functionality. To mitigate the risk of breaking changes affecting CI, we pin the `ruby-all` version when we’re ready to release, [locking it in the `codeql-pack.lock.yml` file](https://docs.github.com/en/code-security/codeql-cli/using-the-advanced-functionality-of-the-codeql-cli/creating-and-working-with-codeql-packs%5C%5C#adding-and-installing-dependencies-on-a-codeql-pack).

Here’s how we manage this setup:

- In our qlpack.yml, we set the dependency to use the latest version of `ruby-all`
- During development, this configuration [pulls in the latest version](https://docs.github.com/en/code-security/codeql-cli/using-the-advanced-functionality-of-the-codeql-cli/creating-and-working-with-codeql-packs#adding-and-installing-dependencies-on-a-codeql-pack%5C)) of `ruby-all` when running `codeql pack init`, ensuring we’re always up to date.
	```
	// Our custom query pack's qlpack.yml
	library: false
	name: github/internal-ruby-codeql
	version: 0.2.3
	extractor: 'ruby'
	dependencies:
	  codeql/ruby-all: "*"
	tests: 'test'
	description: "Ruby CodeQL queries used internally at GitHub"
	```
- Before releasing, we lock the version in the [`codeql-pack.lock.yml`](https://docs.github.com/en/code-security/codeql-cli/using-the-advanced-functionality-of-the-codeql-cli/creating-and-working-with-codeql-packs#codeql-pack-structure) file, specifying the exact version to ensure stability and prevent issues in CI.
	```
	// Our custom query pack's codeql-pack.lock.yml
	lockVersion: 1.0.0
	dependencies:
	 ...
	 codeql/ruby-all:
	   version: 1.0.6
	```

This approach allows us to balance developing against the latest features of the `ruby-all` package while ensuring stability when we release.

We also have a set of [CodeQL unit tests](https://docs.github.com/en/code-security/codeql-cli/using-the-advanced-functionality-of-the-codeql-cli/testing-custom-queries) that exercise our queries against sample code snippets, which helps us quickly determine if any query will cause errors before we publish our pack.

## [Variant analysis](https://github.blog/engineering/how-github-uses-codeql-to-secure-github/?ref=blog.exploits.club&__readwiseLocation=#variant-analysis)

[Variant analysis (VA)](https://codeql.github.com/docs/codeql-overview/about-codeql/#about-variant-analysis) refers to the process of searching for variants of security vulnerabilities. This is particularly useful when we’re responding to a [bug bounty submission](https://bounty.github.com/) or a security incident. We use a combination of tools to do this, including GitHub’s code search functionality, custom scripts, and CodeQL. We will often start by using code search to find patterns similar to the one that caused a particular vulnerability across numerous repositories. This is sometimes not good enough, as code search is not semantically aware, meaning that it cannot determine whether a given variable is an Active Record object or whether it is being used in an \`if\` expression. To answer those types of questions we turn to CodeQL.

## [Writing custom CodeQL queries](https://github.blog/engineering/how-github-uses-codeql-to-secure-github/?ref=blog.exploits.club&__readwiseLocation=#writing-custom-codeql-queries)

### [Tips for getting started](https://github.blog/engineering/how-github-uses-codeql-to-secure-github/?ref=blog.exploits.club&__readwiseLocation=#tips-for-getting-started)

- [CodeQL zero to hero part 1: The fundamentals of static analysis for vulnerability research - The GitHub Blog](https://github.blog/developer-skills/github/codeql-zero-to-hero-part-1-the-fundamentals-of-static-analysis-for-vulnerability-research/)
- [Writing CodeQL queries](https://codeql.github.com/docs/writing-codeql-queries/)
- [Use CodeQL inside Visual Studio Code - GitHub Docs](https://docs.github.com/en/code-security/codeql-for-vs-code)
- [CodeQL workshops for GitHub Universe](https://github.com/githubuniverseworkshops/codeql) and [GitHub Satellite 2020 workshops on finding security vulnerabilities with CodeQL for Java/JavaScript.](https://github.com/githubsatelliteworkshops/codeql)
- [A beginner’s guide to running and managing custom CodeQL queries](https://github.com/readme/guides/custom-codeql-queries)