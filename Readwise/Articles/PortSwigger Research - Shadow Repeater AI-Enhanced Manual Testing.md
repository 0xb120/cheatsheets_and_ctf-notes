---
author: "PortSwigger Research"
aliases: "Shadow Repeater: AI-Enhanced Manual Testing"
tags: RW_inbox, readwise/articles
url: https://portswigger.net/research/shadow-repeater-ai-enhanced-manual-testing
date: 2025-02-26
---
# Shadow Repeater: AI-Enhanced Manual Testing

![rw-book-cover](https://portswigger.net/cms/images/53/94/ffe0-twittercard-shadow_repeater_twitter.png)

## Highlights


Shadow Repeater, which enhances your manual testing with AI-powered, fully automatic variation testing. Simply use Burp Repeater as you normally would, and behind the scenes Shadow Repeater will monitor your attacks, try permutations, and report any discoveries via Organizer.
[View Highlight](https://read.readwise.io/read/01jn1ekbzbcpewyywcv268dmef)



How does Shadow Repeater work
 Shadow Repeater monitors your Repeater requests and identifies which parameters you're changing. It then extracts the payloads you've placed in these parameters, and sends them to an AI model which generates variants. Finally, it attacks the target with these payload variations and uses response diffing to identify whether any of them triggered a new interesting code path.
[View Highlight](https://read.readwise.io/read/01jn1en7e2ga4ghmg6vgy7t2c3)



Usage
 By default, Shadow repeater gets invoked on the 5th repeater request you make, and it requires a parameter or header to be changed.
[View Highlight](https://read.readwise.io/read/01jn1epxxwy130y49mv54m04j4)



When it's found something interesting it will send it to the organiser for inspection.
[View Highlight](https://read.readwise.io/read/01jn1eq8xxmjbgxqabn0s3hqrd)

