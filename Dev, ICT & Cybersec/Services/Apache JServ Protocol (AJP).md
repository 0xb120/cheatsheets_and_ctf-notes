---
Ports:
  - "8009"
Description: According to Apache, AJP (or JK) is a wire protocol. It is an optimized version of the HTTP protocol to allow a standalone web server such as Apache to talk to Tomcat. Historically, Apache has been much faster than Tomcat at serving static content. The idea is to let Apache serve the static content when possible but proxy the request to Tomcat for Tomcat-related content.
---

>[!info]
> According to Apache [^connectors], AJP (or JK) is a wire protocol. It is an optimized version of the HTTP protocol to allow a standalone web server such as Apache to talk to Tomcat. Historically, Apache has been much faster than Tomcat at serving static content. The idea is to let Apache serve the static content when possible but proxy the request to Tomcat for Tomcat-related content.

[^connectors]: [Connectors](https://cwiki.apache.org/confluence/display/TOMCAT/Connectors), cwiki.apache.org

AJP can proxy inbound requests from a web server through to an application server that sits behind the web server. AJP is a highly trusted protocol and should never be exposed to untrusted clients, which could use it to gain access to sensitive information or execute code on the application server.

Read how JSP Message Processing works from the Praetorian blogpost about CVE-2023-46747. [^46747]

[^46747]: [Normal AJP Message Processing](https://www.praetorian.com/blog/refresh-compromising-f5-big-ip-with-request-smuggling-cve-2023-46747/#:~:text=Normal%20AJP%20Message%20Processing), praetorian.com

# Set up a test environment

Setup a test environment:

*tomcat-users.xml*
```xml
<tomcat-users>
  <role rolename="manager-gui"/>
  <role rolename="manager-script"/>
  <user username="tomcat" password="s3cret" roles="manager-gui,manager-script"/>
</tomcat-users>
```

Install tomcat using Docker:
```bash
$ sudo apt install docker.io
$ sudo docker run -it --rm -p 8009:8009 -v `pwd`/tomcat-users.xml:/usr/local/tomcat/conf/tomcat-users.xml --name tomcat "tomcat:8.0"
```
# Exploitation

When we come across open AJP proxy ports (`8009 TCP`) during penetration tests, we may be able to use them to access the "hidden" Apache Tomcat Manager behind it. We can configure our own Nginx or Apache webserver with AJP modules to interact with it and access the underlying application.

## Nginx & AJP

We can use Nginx with the `ajp_module` to access the "hidden" Tomcat Manager. This can be done by compiling the Nginx source code and adding the required module:

```bash
$ wget https://nginx.org/download/nginx-1.21.3.tar.gz
$ tar -xzvf nginx-1.21.3.tar.gz
$ git clone https://github.com/dvershinin/nginx_ajp_module.git
$ cd nginx-1.21.3
$ sudo apt install libpcre3-dev
$ ./configure --add-module=`pwd`/../nginx_ajp_module --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib/nginx/modules
$ make
$ sudo make install
$ nginx -V
```

Edit the `/etc/nginx/conf/nginx.conf`:
```nginx
# Comment out the entire "server" block and append inside http
	http {
	# server {
	# ...
	# }
	upstream tomcats {
		server <TARGET_SERVER>:8009; # AJP port
		keepalive 10;
	}
	server {
		listen 80;
		location / {
			ajp_keep_conn on;
			ajp_pass tomcats;
		}
	}
}
```

## Apache & AJP

Apache has the AJP module precompiled for us. We will need to install it, though, as it doesn't come in default installations.

```bash
sudo apt install libapache2-mod-jk
sudo a2enmod proxy_ajp
export TARGET="<TARGET_IP>"
echo -n """<Proxy *>
Order allow,deny
Allow from all
</Proxy>
ProxyPass / ajp://$TARGET:8009/
ProxyPassReverse / ajp://$TARGET:8009/""" | sudo tee /etc/apache2/sites-available/ajp-proxy.conf
sudo ln -s /etc/apache2/sites-available/ajp-proxy.conf /etc/apache2/sites-enabled/ajp-proxy.conf
sudo systemctl start apache2
```
