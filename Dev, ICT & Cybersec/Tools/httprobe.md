---
Description: Take a list of domains and probe for working http and https servers.
URL: https://github.com/tomnomnom/httprobe
---

>[!summary]
>Take a list of domains and probe for working http and https servers.

### Usage 

Basic usage:
```bash
echo '0xbro.red
sicuranext.com
asdfwfe.fceshfnuewkur.org' | httprobe
http://0xbro.red
http://sicuranext.com
https://sicuranext.com
https://0xbro.red
```


#### Find HTTP services hosted on non-standard ports

Find other services on non-standard ports:
```bash
mattia_m@SNITNB00021MB ~ % echo 151.0.207.202 | httprobe

mattia_m@SNITNB00021MB ~ % echo 151.0.207.202 | httprobe -p 'http:8080'
http://151.0.207.202:8080
```