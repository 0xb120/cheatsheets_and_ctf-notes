---
raindrop_id: 1321645826
raindrop_highlights:
  68d2a07d23c49007267fa9a1: f80e6e13a3b93ae71077be36e7e91d05
  68d2a09cb16bc7033c43f560: 9b33da6bae514bc058f9303ffdc7d8b5
  68d2a0a9a2be5671f27ecd16: 4f4df6754572242cc3f5763771532af2
  68d2a0da3640ed9ab6fac9e7: 77626099880d380b360fd5359d178f35
  68d2a0e8a2be5671f27ed610: b1d777b3dece7e598c552ec6a7be6b77
  68d2a0ec0e39366081db853c: e26a834399dacc4d7742eb32a7f7ecc7
  68d2a0f40e39366081db8694: 33e73bb7c5df8a2a68fffaf8657394d8
  68d2a0f75430135b7fe0451d: 59f5b61281555c951d7cc2093c00bdaf
  68d2a10e52f45e8876c1d058: b3b56c057ce0a6801b635271883119e0
  68d2a1120d9bc9bcb7b831b4: bd658fa9795a074e7229c663eaaa0a6a
  68d2a1305430135b7fe04e4f: bd27262c73c3ce2ae7d9b815860ab623
title: "Referral Beware, Your Rewards are Mine (Part 1)"

description: |-
  Referral rewards programs are nearly ubiquitous today, from consumer tech to SaaS companies, but are rarely given much security oversight.

source: https://rhinosecuritylabs.com/research/referral-beware-your-rewards-are-mine-part-1

created: 1756314192000
type: article
tags:
  - "_index"

 
  - "tech-blog"

---
# Referral Beware, Your Rewards are Mine (Part 1)

![](https://rhinosecuritylabs.com/wp-content/uploads/2025/08/image-1024x688.png)

> [!summary]
> Referral rewards programs are nearly ubiquitous today, from consumer tech to SaaS companies, but are rarely given much security oversight.





The Common Pitfalls of Referral Rewards Implementations
Client-Side Gadgets

Applications often will have functionality or bugs that are either of little impact, or aren’t a vulnerability by themselves, but when used in conjunction with other functionality or low impact bugs, can be used to achieve a serious vulnerability.
Within the referral program functionality, two gadgets can be found:

Cookie Injection
Client-Side Path Traversal
Business Logic Flaws
The most common bug type found in these implementations were some form of business logic flaw. The implementation logic within these various programs made various assumptions on how the user would interact with the program and this led to various instances where the intended flow could be circumvented. Some areas where assumptions might occur can include:

Validation of the referral code or other validation requests/flows
That users will follow the intended flow
User will only enter expected input
Race Condition
It is also possible in referral rewards implementations for a user to get multiple rewards for a single code redemption if a race condition exists in the request that applies the code.
Referral Hijacking
Referral hijacking is a new attack vector found during this research that encompasses the ability for an attacker to change the referral link/code that is sent to a prospective user to the attacker link/code, thus hijacking the referral and becoming the receiver of the reward given to the referrer.
One attack path to referral hijacking was discovered: cookie fixation.
As mentioned in the cookie-injection gadget section, an attacker can fixate a cookie for a victim by setting the path cookie attribute to a specific path. According to RFC 6265, when a browser finds multiple cookies with the same name, the cookie with the more specific path gets prioritized and sent first. This means that in situations where the referral program is using the URL to Cookie implementation, an attacker with the ability to fixate a cookie can fixate the referral cookie containing the attacker referral code and set the path to the request that applies the referral code, which will lead to the attacker hijacking the referral and receiving the reward instead of the original referring user.