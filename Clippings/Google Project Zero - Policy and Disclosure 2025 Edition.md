---
title: "Policy and Disclosure: 2025 Edition"
source: "https://googleprojectzero.blogspot.com/2025/07/reporting-transparency.html"
author:
  - "Google Project Zero"
published: 2025-07-29
created: 2025-08-19
description: "Posted by Tim Willis, Google Project Zero In 2021, we updated our vulnerability disclosure policy to the current \"90+30\" model. Our goals we..."
tags: ["clippings/articles", "_inbox"]
---
# Policy and Disclosure: 2025 Edition

![]()

> [!summary]
> > Google Project Zero announced a new \"Reporting Transparency\" trial policy, effective July 29, 2025, to address the \"upstream patch gap\" – the delay between an upstream vendor fixing a vulnerability and downstream products integrating it.
> The core \"90-day disclosure deadline\" remains, but within approximately one week of reporting a vulnerability, Project Zero will publicly share: the vendor/project, affected product, report date, and the 90-day disclosure deadline.
> The goal is to increase transparency for downstream dependents, encouraging better communication and faster patch adoption for end-users, without releasing technical details that could aid attackers before the deadline.

### Policy and Disclosure: 2025 Edition

In 2021, we updated our vulnerability disclosure policy to the current "90+30" model. Our goals were to drive faster yet thorough patch development, and improve patch adoption. While we’ve seen progress, a significant challenge remains: the time it takes for a fix to actually reach an end-user's device.

our work has highlighted a critical, earlier delay: the "upstream patch gap". This is the period where an upstream vendor has a fix available, but downstream dependents, who are ultimately responsible for shipping fixes to users, haven’t yet integrated it into their end product.

For the end user, a vulnerability isn't fixed when a patch is released from Vendor A to Vendor B; it's only fixed when they download the update and install it on their device. To shorten that entire chain, we need to address the upstream delay.

To address this, we're announcing a new trial policy: Reporting Transparency.

#### The Trial: Reporting Transparency

Our core [90-day disclosure deadline](https://googleprojectzero.blogspot.com/p/vulnerability-disclosure-policy.html) will remain in effect. However, we're adding a new step at the beginning of the process.

Beginning today, within approximately one week of reporting a vulnerability to a vendor, we will [publicly share](https://googleprojectzero.blogspot.com/p/reporting-transparency.html) that a vulnerability was discovered. We will share:

The vendor or open-source project that received the report.

The affected product.

The date the report was filed, and when the 90-day disclosure deadline expires.