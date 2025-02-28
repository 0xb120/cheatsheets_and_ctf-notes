HTML is considered a tolerant language because of its forgiving nature when it encounters errors or unexpected code. Unlike some stricter programming languages, HTML prioritizes displaying content even if the code isn't perfectly written. [](https://read.readwise.io/read/01jn6dce59zymfqjmsr6xzeaxw)

When a broken markup is rendered, instead of crashing or displaying an error message, browsers attempt to interpret and fix the HTML as best as they can, even if it contains minor syntax errors or missing elements. [](https://read.readwise.io/read/01jn6ddkkf4tc0a0sm62h7k7c0)

>[!example]
>Opening the following markup in the browser `<p>test` will execute as expected despite missing a closing `p` tag. When looking at the final page’s HTML code we can see that the parser fixed our broken markup and closed the `p` element by itself: `<p>test</p>`. [](https://read.readwise.io/read/01jn6de0b35ghyn531d978gjc7)

But how does our HTML parser know in which way to “fix” a broken markup? 
Should `<a><b>` become`<a></a><b></b>` or `<a><b></b></a>`? 

To answer this question there is a well-documented [HTML specification](https://html.spec.whatwg.org/), but unfortunately, there are still some ambiguities that result in different HTML parsing behaviors even between major browsers today. [](https://read.readwise.io/read/01jn6dexptcjwxnnzbmxjee105)

## Mutation

Mutation in HTML is any kind of change made to the markup for some reason or another.
 - When a parser fixes a broken markup (`<p>test` → `<p>test</p>`), that's a mutation. 
 - Normalizing attribute quotes (`<a alt=test>` → `<a alt=”test”>`), that's a mutation.
 - Rearranging elements (`<table><a>` → `<a></a><table></table>`), that's a mutation
 - And so on…

## HTML Parsing Background

### Different content parsing types

 HTML isn't a one-size-fits-all parsing environment. Elements handle their content differently, with seven distinct [parsing modes](https://html.spec.whatwg.org/#elements-2) at play: [](https://read.readwise.io/read/01jn6djhg40ea7fqdc4b034ejr)

- [void elements](https://html.spec.whatwg.org/#void-elements)
	- `area`, `base`, `br`, `col`, `embed`, `hr`, `img`, `input`, `link`, `meta`, `source`, `track`, `wbr`
 - [the `template` element](https://html.spec.whatwg.org/#the-template-element-2)
	 - `template`
 - [Raw text elements](https://html.spec.whatwg.org/#raw-text-elements)
	 - `script`, `style`, `noscript`, `xmp`, `iframe`, `noembed`, `noframes`
 - [Escapable raw text elements](https://html.spec.whatwg.org/#escapable-raw-text-elements)
	 - `textarea`, `title`
 - [Foreign content elements](https://html.spec.whatwg.org/#foreign-elements)
	 - `svg`, `math`
 - [Plaintext state](https://html.spec.whatwg.org/#plaintext-state)
	 - `plaintext`
 - [Normal elements](https://html.spec.whatwg.org/#normal-elements)
	 - All other allowed HTML elements are normal elements.

We can fairly easily demonstrate a difference between parsing types using the following example:
 1. Our first input is a `div` element, which is a “normal element” element: 
	 `<div><a alt="</div><img src=x onerror=alert(1)>">`
 2. On the other hand, the second input is a similar markup using the `style` element instead (which is a “raw text”): 
	 `<style><a alt="</style><img src=x onerror=alert(1)>">`
 
 Looking at the parsed markup we can clearly see the parsing differences:

![](attachments/HTML-1.png)
![](attachments/HTML-2.png)

#### Foreign content elements

HTML5 introduced new ways to integrate specialized content within web pages. Two key examples are the `<svg>` and `<math>` elements. These elements leverage distinct namespaces, meaning they follow different parsing rules compared to standard HTML.

Eg. `<svg><style><a alt="</style><img src=x onerror=alert(1)>">`

![](attachments/HTML-3.png)

In this case, we do see an `a` element being created. The `style` element doesn’t follow the “raw text” parsing rules, because it is inside a different namespace. When residing within an SVG or MathML namespace, the parsing rules change and no longer follow the HTML language. [](https://read.readwise.io/read/01jn6dya819r3jzwxm5xk10cwg)