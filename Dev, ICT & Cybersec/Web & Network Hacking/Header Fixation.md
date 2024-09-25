---
aliases:
  - Cookie Fixation
---
# Header Fixation

Header fixation happens when the server is configured in order to reflect request's header in server responses. This issue open to a variety of vulnerabilities, including [Cookie Fixation](Header%20Fixation.md#Cookie%20Fixation)

# Cookie Fixation

Cookie fixation is a particular vulnerability, some time originating from header fixation, that allows attacker to fixate arbitrary cookie values inside victims browsers.

>[!danger]
>The capability to **set a cookie in a more specific path** is very interesting as you will be able to make the **victim work with his cookie except in the specific path where the malicious cookie set will be sent before**.

This can be a useful gadget for different kind of vulnerabilities:
- [Session Fixation](Session%20Fixation.md)
- [Cookie Tossing](Cookie%20Tossing.md) (Cookie Fixation on parent domain)

# Potential bypass to cookie protections (?)

![Bypass using nameless cookie](Cookie%20Tossing.md#Bypass%20using%20nameless%20cookie)