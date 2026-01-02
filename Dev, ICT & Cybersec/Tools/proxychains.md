---
Description: 'A tool that forces any TCP connection made by any given application to follow through proxy like TOR or any other SOCKS4, SOCKS5 or HTTP(S) proxy. Supported auth-types: "user/pass" for SOCKS4/5, "basic" for HTTP.'
URL: https://github.com/haad/proxychains
---

```bash
$ proxychains4                                        

Usage:  proxychains4 -q -f config_file program_name [arguments]
        -q makes proxychains quiet - this overrides the config setting
        -f allows one to manually specify a configfile to use
        for example : proxychains telnet somehost.com
More help in README file
```

## Files

>[!info]
>proxychains looks for config file in following order:
>1. file listed in environment variable `PROXYCHAINS_CONF_FILE` or provided as a -f argument to proxychains script or binary.
>2. `./proxychains.conf`
>3. `$(HOME)/.proxychains/proxychains.conf`
>4. `/etc/proxychains.conf`
