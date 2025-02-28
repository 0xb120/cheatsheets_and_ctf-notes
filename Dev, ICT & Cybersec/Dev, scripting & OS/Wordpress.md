# External Resources

- [Most Common WordPress Vulnerabilities & How to Fix Them](https://patchstack.com/articles/common-plugin-vulnerabilities-how-to-fix-them/)
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
## Easy Wordpress setup

```yaml title:wp-compose.yaml
services:
  db:
    # We use a mariadb image which supports both amd64 & arm64 architecture
    image: mariadb:10.6.4-focal
    # If you really want to use MySQL, uncomment the following line
    #image: mysql:8.0.27
    command: '--default-authentication-plugin=mysql_native_password'
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    expose:
      - 3306
      - 33060

  wordpress:
    image: wordpress:latest
    ports:
      - 8080:80
    restart: always
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DEBUG: 1
      XDEBUG_MODE: debug
      XDEBUG_CONFIG: client_host=172.17.0.1 client_port=9003
      PHP_EXTENSION_XDEBUG: 1
    volumes:
      - ./code:/var/www/html:rw
volumes:
  db_data:
```

- [Debugging WordPress Using Xdebug, Local, and VS Code](https://webdevstudios.com/2022/10/06/debugging-wordpress/)
- https://github.com/patchstack/wp-xdebug-docker: A fully debuggable WordPress docker instance with XDebug installed
- [dimasma0305/dockerized-wordpress-debug-setup](https://github.com/dimasma0305/dockerized-wordpress-debug-setup): This project is a Dockerized WordPress development environment with two configurations, one using Nginx and the other using Apache. It includes Xdebug for debugging purposes and provides flexibility for developers to choose their preferred web server.
- [Wordpress - Getting Started](../../../Readwise/Articles/Alex%20Thomas%20-%20How%20to%20Find%20XSS%20(Cross-Site%20Scripting)%20Vulnerabilities%20in%20WordPress%20Plugins%20and%20Themes.md#Getting%20Started)

# Worpress in notes

```query
tag:wordpress
```
