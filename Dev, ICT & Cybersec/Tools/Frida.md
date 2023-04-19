---
Description: Dynamic instrumentation toolkit for developers, reverse-engineers, and security researchers.
URL: https://frida.re/
---

>[!summary] 
>It’s [Greasemonkey](https://addons.mozilla.org/en-US/firefox/addon/greasemonkey/) for native apps, or, put in more technical terms, it’s a dynamic code instrumentation toolkit. It lets you inject snippets of JavaScript or your own library into native apps on Windows, macOS, GNU/Linux, iOS, Android, and QNX. Frida also provides you with some simple tools built on top of the Frida API. These can be used as-is, tweaked to your needs, or serve as examples of how to use the API.

# Documentation and CodeShare

- https://frida.re/docs/home/
- https://codeshare.frida.re/

## Frida server

>[!info] Download
>https://github.com/frida/frida/releases

```bash
adb push ..\..\frida\frida-server-16.0.11-android-x86 /data/local/tmp
adb shell
su
chmod +x /data/local/tmp/frida-server-16.0.11-android-x86
/data/local/tmp/frida-server-16.0.11-android-x86

netstat -tulnp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program Name
tcp        0      0 127.0.0.1:27042         0.0.0.0:*               LISTEN      7846/frida-server-16.0.11-android-x86
```

## Expose the frida server on a public port

```bash
/data/local/tmp/frida-server-16.0.11-android-x86 -l 0.0.0.0:27042
 # netstat -tulnp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program Name
tcp        0      0 0.0.0.0:27042           0.0.0.0:*               LISTEN      4990/frida-server-16.0.11-android-x86
```