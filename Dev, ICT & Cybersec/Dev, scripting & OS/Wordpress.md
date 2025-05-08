# Wordpress Introduction

- [WordPress for Security Audit](../../Readwise/Articles/Antoine%20Gicquel%20-%20WordPress%20for%20Security%20Audit.md)
- [WordPress Request Architecture and Hooks](../../Readwise/Articles/Alex%20Thomas%20-%20WordPress%20Security%20Research%20Series%20WordPress%20Request%20Architecture%20and%20Hooks.md)

WP official documentation:

- [Plugin Handbook](https://developer.wordpress.org/plugins/)
- [Developer Resources](https://developer.wordpress.org/)
	- [Rest API Handbook](https://developer.wordpress.org/rest-api/)
	- [api.wordpress.org/plugins/](https://api.wordpress.org/plugins/info/1.2/?action=query_plugins&request[page]=1&request[per_page]=10&request[search]=&request[author]=&request[tag]=)

## Easy setup

- https://github.com/0xb120/wp-plugin-downlauditor
- [Debugging WordPress Using Xdebug, Local, and VS Code](https://webdevstudios.com/2022/10/06/debugging-wordpress/)
- https://github.com/patchstack/wp-xdebug-docker: A fully debuggable WordPress docker instance with XDebug installed
- [dimasma0305/dockerized-wordpress-debug-setup](https://github.com/dimasma0305/dockerized-wordpress-debug-setup): This project is a Dockerized WordPress development environment with two configurations, one using Nginx and the other using Apache. It includes Xdebug for debugging purposes and provides flexibility for developers to choose their preferred web server.
- [Wordpress - Getting Started](../../../Readwise/Articles/Alex%20Thomas%20-%20How%20to%20Find%20XSS%20(Cross-Site%20Scripting)%20Vulnerabilities%20in%20WordPress%20Plugins%20and%20Themes.md#Getting%20Started)

## Vulnerability research on Wordpress and plugins

- [How to Find XSS (Cross-Site Scripting) Vulnerabilities in WordPress Plugins and Themes](../../Readwise/Articles/Alex%20Thomas%20-%20How%20to%20Find%20XSS%20(Cross-Site%20Scripting)%20Vulnerabilities%20in%20WordPress%20Plugins%20and%20Themes.md)
- [CVE Hunting Made Easy](../../Readwise/Articles/Eddie%20Zhang,%20Aug%2027%20-%20CVE%20Hunting%20Made%20Easy.md)

Pre-made tools:
- [WPProbe](../../Readwise/Articles/httpsgithub.comChocapikk%20-%20GitHub%20-%20ChocapikkWpprobe%20A%20Fast%20WordPress%20Plugin%20Enumeration%20Tool.md)
- [wpscan](https://github.com/wpscanteam/wpscan)

### Code mirroring and listing

- [Plugin SVN](https://plugins.svn.wordpress.org/): it contains the source code for all the plugins that are available at [https://wordpress.org/plugins](https://wordpress.org/plugins)
	- /tags/ lists the plugins' source code by minor versions
	- /trunks/ has the latest development code in the plugin
- [Themese SSVN](https://themes.svn.wordpress.org/): all the themes hosted at https://wordpress.org/themes
- [Wordpress Trac](https://plugins.trac.wordpress.org/browser/elementor): is primarily used to track down all the code changes in the plugin/theme. It's like a public Git repo listing commits and changes to the code. Selecting two different commits and clicking on “View changes” displays all the changes within the plugin files.

### Code search

- [WP Directory](https://wpdirectory.net/): mirrors the WordPress Plugin and Theme Directories, allowing lightning fast regex search.



## Cheatsheet and other resources

- [WordPress Plugin Security Testing Cheat Sheet](https://github.com/wpscanteam/wpscan/wiki/WordPress-Plugin-Security-Testing-Cheat-Sheet): source, sinks and useful #Wordpress setup options, by wpscan
- [Most Common WordPress Vulnerabilities & How to Fix Them](https://patchstack.com/articles/common-plugin-vulnerabilities-how-to-fix-them/)
- PatchStack Academy
	- https://patchstack.com/academy/wordpress/vulnerabilities/
	- [patchstack/vulnerability-playground](https://github.com/patchstack/vulnerability-playground): This plugin designed to contain vulnerable function or process to be used to exercise on hacking WordPress ecosystem. Code reference from Patchstack Academy site.
	- [Patchstack academy with focus on Wordpress](https://patchstack.com/academy/wordpress/getting-started/)
		- [Functions](https://patchstack.com/academy/wordpress/wordpress-internals/functions/)
			- [`current_user_can`](https://patchstack.com/academy/wordpress/wordpress-internals/functions/#current_user_can)
			- [`wp_verify_nonce`](https://patchstack.com/academy/wordpress/wordpress-internals/functions/#wp_verify_nonce)
			- [`check_admin_referer`](https://patchstack.com/academy/wordpress/wordpress-internals/functions/#check_admin_referer)
			- [`check_ajax_referer`](https://patchstack.com/academy/wordpress/wordpress-internals/functions/#check_ajax_referer)
			- [`register_rest_route`](https://patchstack.com/academy/wordpress/wordpress-internals/functions/#register_rest_route)
		- [Hooks](https://patchstack.com/academy/wordpress/wordpress-internals/hooks/)
			- [`init`](https://patchstack.com/academy/wordpress/wordpress-internals/hooks/#init)
			- [`admin_init`](https://patchstack.com/academy/wordpress/wordpress-internals/hooks/#admin_init)
			- [`wp_ajax_{$action}`](https://patchstack.com/academy/wordpress/wordpress-internals/hooks/#wp_ajax_action)
			- [`wp_ajax_nopriv_{$action}`](https://patchstack.com/academy/wordpress/wordpress-internals/hooks/#wp_ajax_nopriv_action)
		- [shortcodes](https://codex.wordpress.org/Shortcode_API)
- [GitHub - Chocapikk/Wpprobe: A Fast WordPress Plugin Enumeration Tool](../../Readwise/Articles/httpsgithub.comChocapikk%20-%20GitHub%20-%20ChocapikkWpprobe%20A%20Fast%20WordPress%20Plugin%20Enumeration%20Tool.md)



# Worpress in notes

```query
tag:wordpress
```
