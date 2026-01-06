---
author: Nathan Touati
aliases:
  - "Sandwich Attacks: From Reset Password to Account Takeover"
  - Sandwich Attack
tags:
  - readwise/articles
url: https://appsec-labs.com/sandwich-attacks/
created: 2025-01-23
---
# Sandwich Attacks: From Reset Password to Account Takeover

![rw-book-cover](https://appsec-labs.com/wp-content/uploads/2024/12/sandwich.png)

## Highlights

# Time-based vs. Random: Unique Doesn’t Mean Secure

“unique” isn’t always the same as “unpredictable.” If the token generation relies on something like the server’s clock rather than randomness, a clever attacker could guess the victim’s reset link and hijack their account. This sneaky tactic is known as a Sandwich Attack.

Many systems use **UUIDv1** to generate tokens. UUIDv1 is time-based, meaning it incorporates the current timestamp plus a hardware MAC address to produce something unique for that moment. The upside? Two tokens generated at slightly different times are never the same. The downside? They’re not truly random and can be predicted if someone understands how they’re formed.
> [View Highlight](https://read.readwise.io/read/01jj96n0ynkgfn2zd6k994m0qr)


# The Sandwich Attack Explained

 Here’s how an attacker can exploit predictable time-based UUIDs during a password reset:
 1. **First Request (Attacker’s Email):** The attacker triggers a password reset for their own account, obtaining a UUIDv1 token. For example: `https://example.com/reset-password?token=6f3da648-bc98-11ef-9cd2-0242ac120002`
 2. **Second Request (Victim’s Email):** Immediately after, the attacker triggers a password reset for the victim’s account. Another UUIDv1 token is generated, close to the first one in time: `https://example.com/reset-password?token=6f3da7d8-bc98-11ef-9cd2-0242ac120002`
 3. **Third Request (Attacker’s Email Again):** The attacker then triggers another reset for their own account, receiving a third token, again closely aligned with the victim’s: `https://example.com/reset-password?token=6f3da882-bc98-11ef-9cd2-0242ac120002`

 These three tokens form a “sandwich” around the victim’s token—attacker, victim, attacker. Notice how the tokens differ only slightly at the end. Because the UUIDs are time-based, all three are very similar. By knowing the tokens before and after the victim’s, the attacker can guess what the victim’s token might be, simply by generating more UUIDs from the same timeframe, until they find the one that matches the victim’s reset link.
> [View Highlight](https://read.readwise.io/read/01jj96p473dpe2phkadsf7mhtp)

