The Regular expression Denial of Service (ReDoS) is a Denial of Service attack, that exploits the fact that most Regular Expression implementations may reach extreme situations that cause them to work very slowly. [^redos]

[^redos]: [Regular Expression Denial of Service](https://owasp.org/www-community/attacks/Regular_expression_Denial_of_Service_-_ReDoS), owasp.org

ReDoS can be both a client side attack and a server side attack, depending on which side the flawed regex is applied.

Due to the way that many regular expression engines work it is possible to write an expression that, with the right input, will cause the engine to take a long time to evaluate. In JavaScript, this will occupy the main thread and halt the event loop until the expression has been completely evaluated. 

>[!example] Stack Overflow Example
>In the Stack Overflow outage, the offending regular expression was `/^[\s\u200c]+|[\s\u200c]+$/`

Put together, the expression looks for one or more whitespace characters at the start of a line or one or more whitespace characters at the end of a line. It was being used to trim whitespace from the beginning or end of a line. This works great if the string begins or ends with a whitespace character. However, if a string ends with a lot of space characters and then a non-space character it will cause an issue. [^ref]

The reason is that the engine backtracks to where it started the match, discards the first " " and starts again with the second " ". It match for a while and then fails again, backtracking another time, again and again.

[^ref]: [Phil Nash - A Comprehensive Guide to the Dangers of Regular Expressions in JavaScript](../../Readwise/Articles/Phil%20Nash%20-%20A%20Comprehensive%20Guide%20to%20the%20Dangers%20of%20Regular%20Expressions%20in%20JavaScript.md#^ee519f), regex 101

Other similar examples can be found in node-fetch [^node-fetch], Cloudflare [^cloudflare], etc.

[^node-fetch]: [Phil Nash - A Comprehensive Guide to the Dangers of Regular Expressions in JavaScript](../../Readwise/Articles/Phil%20Nash%20-%20A%20Comprehensive%20Guide%20to%20the%20Dangers%20of%20Regular%20Expressions%20in%20JavaScript.md#^f3112b), node-fetch
[^cloudflare]: [Cloudflare outage on July 2, 2019](https://blog.cloudflare.com/details-of-the-cloudflare-outage-on-july-2-2019/), blog.cloudflare.com

Dangerous regexes:
- `/\s*,\s*/` 
- `/^(.+\.)*localhost$/`
- `/^[\s\u200c]+|[\s\u200c]+$/`
- `(?:(?:\"|'|\]|\}|\\|\d|(?:nan|infinity|true|false|null|undefined|symbol|math)||\-|\+)+[)]*;?((?:\s|-|~|!|{}|\|\||\+)*.*(?:.*=.*)))`