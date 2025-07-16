---
Description: httpx is a fast and multi-purpose HTTP toolkit that allows running multiple probes using the retryablehttp library.
URL: https://github.com/projectdiscovery/httpx
---

>[!summary] 
`httpx` is a fast and multi-purpose HTTP toolkit that allows running multiple probes using the [retryablehttp](https://github.com/projectdiscovery/retryablehttp-go) library. It is designed to maintain result reliability with an increased number of threads.

### Usage

#### Find HTTP services hosted on non-standard ports

```bash
echo 151.0.207.202 | httpx -silent


echo 151.0.207.202 | httpx -silent -p http:8080,8000-8010
http://151.0.207.202:8000
http://151.0.207.202:8002
http://151.0.207.202:8006
http://151.0.207.202:8007
http://151.0.207.202:8005
http://151.0.207.202:8008
http://151.0.207.202:8003
http://151.0.207.202:8001
http://151.0.207.202:8080
```