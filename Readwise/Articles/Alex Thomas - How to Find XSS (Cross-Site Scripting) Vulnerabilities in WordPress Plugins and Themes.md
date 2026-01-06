---
author: Alex Thomas
aliases:
  - How to Find XSS (Cross-Site Scripting) Vulnerabilities in WordPress Plugins and Themes
tags:
  - readwise/articles
  - code-review
  - wordpress
  - wordpress/plugins
url: https://www.wordfence.com/blog/2024/09/how-to-find-xss-cross-site-scripting-vulnerabilities-in-wordpress-plugins-and-themes/?__readwiseLocation=
created: 2024-09-06
---
# How to Find XSS (Cross-Site Scripting) Vulnerabilities in WordPress Plugins and Themes

### Methodology
  Successful discovery of [Cross-Site Scripting (XSS)](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Cross-Site%20Scripting%20(XSS).md) vulnerabilities requires a solid and repeatable research methodology. The most efficient methodology is to take a white-box approach and analyze the source code of various plugins and themes while following the steps below:
  1. Search for sources and make note of them
  2. Search for sinks and make note of them
  3. Map the data flow from source to sink OR sink to source
  4. Make note of any modifications or transformations of input data from source to sink or vice versa
	  1. Does the input run through a validation, sanitization, escaping, or decoding function?
		  1. Is it implemented correctly?
	  2. Is the input modified by other functions – what elements are added or removed?
  5. How is the output echoed or printed?
	  1. Is the output placed within an HTML tag, a tag attribute, or between other tags?
	  2. How is the output concatenated with other strings? Are they quoted correctly?
  6. How does the output look in the HTML of a rendered page when a special character, such as single quote (`‘`), or an HTML tag, such as `<script` is passed?

  Alternatively, you can use a black-box approach of installing various plugins/themes and then analyzing them while interacting with them as a user. Considering you will always have access to the source code for WordPress plugins and themes, we don’t recommend taking this approach from an efficiency standpoint though it is definitely a great way to get started and get familiar with the functionality of a plugin or theme before diving into the code analysis. [](https://read.readwise.io/read/01j7398ayq5r3zn286wnq3sp9f)

### Common XSS Sources

| PHP Superglobals                                                          | Database Inputs                                                                        | Third-Party Integrations | Alternative Input Streams                                                 |
| ------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- | ------------------------ | ------------------------------------------------------------------------- |
| [$_POST](https://www.php.net/manual/en/reserved.variables.post.php)       | [Options Table](https://codex.wordpress.org/Database_Description#Table:_wp_options)    | External APIs            | [php://](https://www.php.net/manual/en/wrappers.php.php)                  |
| [$_GET](https://www.php.net/manual/en/reserved.variables.get.php)         | [User Meta Table](https://codex.wordpress.org/Database_Description#Table:_wp_usermeta) | Webhooks                 | [filter_input()](https://www.php.net/manual/en/function.filter-input.php) |
| [$_REQUEST](https://www.php.net/manual/en/reserved.variables.request.php) | [Post Meta Table](https://codex.wordpress.org/Database_Description#Table:_wp_postmeta) | Embedded Content         |                                                                           |
| [$_COOKIE](https://www.php.net/manual/en/reserved.variables.cookies.php)  | Custom Tables                                                                          | RSS Feeds                |                                                                           |
| [$_FILES](https://www.php.net/manual/en/reserved.variables.files.php)     |                                                                                        |                          |                                                                           |
| [$_SERVER](https://www.php.net/manual/en/reserved.variables.server.php)   |                                                                                        |                          |                                                                           |
| [$_SESSION](https://www.php.net/manual/en/reserved.variables.session.php) |                                                                                        |                          |                                                                           |
[](https://read.readwise.io/read/01j73996wqt7qcr2ba8ew2t6p6)


You can use an Integrated Development Environment (IDE), a text editor, or command-line tools like `grep` to find **sources** in WordPress plugin or theme code with a simple regular expression such as the following:  [](https://read.readwise.io/read/01j739ayxhfzfnc77qez7tqbxc)
`\$_(GET|POST|REQUEST|COOKIE|FILES|SERVER|SESSION)\b|\b(get_option|get_user_meta|get_post_meta|get_comment_meta|get_site_option|get_network_option|apply_filters|do_action)\b`


You can also search for **sinks** using a regular expression like in the following example:
`\b(echo|print|printf|sprintf|die|wp_die)\s*\(`
 [](https://read.readwise.io/read/01j739bxb8g9zbawqk8hcvb2ne)

### Getting Started
  To get started, we recommend joining the [Wordfence Discord](https://discord.com/invite/awPVjTNTrn) channel and downloading the Docker configuration for a WordPress local test site from the [resources](https://discordapp.com/channels/1197901373581303849/1199013923173712023) channel. The configuration, as well as the installation instructions, can be found in a pinned post at the top of the channel.
  
  This Docker configuration will provide a local test environment that includes WordPress, XDebug (along with a debugging configuration for Visual Studio Code), Adminer (a web-based GUI for database operations), Mailcatcher (a web based mail interface for intercepting mail sent by the WordPress instance), and WordPress CLI (a command-line interface for WordPress). We also recommend reviewing part one of our beginner research series [to learn more about WordPress architecture in general](https://www.wordfence.com/blog/2024/07/wordpress-security-research-series-wordpress-request-architecture-and-hooks/).
 [](https://read.readwise.io/read/01j739j7sm5wspntdjy45zh5dx)

