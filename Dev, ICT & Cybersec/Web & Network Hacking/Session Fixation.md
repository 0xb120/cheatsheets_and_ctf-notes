Session Fixation occurs when an attacker can fixate a (valid) session identifier. The attacker will then have to trick the victim into logging into the application using the aforementioned session identifier. If the victim does so, the attacker can proceed to a Session Hijacking attack (since the session identifier is already known).

Such bugs usually occur when session identifiers (such as cookies) are being accepted from _URL Query Strings_ or _Post Data_, but it can happens also from a Local/Physical attack vector.

1. Attacker manages to obtain a valid session identifier
2. Attacker manages to fixate a valid session identifier
	- The assigned session identifier pre-login remains the same post-login and
	- Session identifiers (such as cookies) are being accepted from URL Query Strings or Post Data and propagated to the application
3. Attacker tricks the victim into establishing a session using the abovementioned session identifier

Example:

The HTTP request below set the session cookie with the value provided in the `token` HTTP parameter, which is under user control:
```http
GET /?redirect_uri=/complete.html&token=<RANDOM-TOKEN-VALUE> HTTP/1.1
Host: vulnerable.com

--- RESPONSE ---

HTTP/1.1 200 OK
Cookie: PHPSESSIONID=<RANDOM-TOKEN-VALUE>
```

Then, after a successful login, the cookie is not renew but only refreshed:
```http
POST /login HTTP/1.1
Host: vulnerable.com
Cookie: PHPSESSIONID=<RANDOM-TOKEN-VALUE>

user=foo&password=bar

--- SEPARATOR ---

HTTP/1.1 200 OK
Cookie: PHPSESSIONID=<SAME-RANDOM-TOKEN-VALUE>
```

In this scenario, an attacker can set an arbitrary cookie value tricking the user into clicking a custom link containing it's cookie. Then, after a successful login made by the victim, the attacker can use the same cookie provided to the victim in order to impersonate it.