---
author: "blackhat.com"
aliases: "Us-17-Lekies-Dont-Trust-the-Dom-Bypassing-XSS-Mitigations-via-Script-Gadgets"
tags: RW_inbox, readwise/articles
url: https://www.blackhat.com/docs/us-17/thursday/us-17-Lekies-Dont-Trust-The-DOM-Bypassing-XSS-Mitigations-Via-Script-Gadgets.pdf
date: 2025-01-08
---
# Us-17-Lekies-Dont-Trust-the-Dom-Bypassing-XSS-Mitigations-via-Script-Gadgets

![rw-book-cover](https://readwise-assets.s3.amazonaws.com/media/reader/parsed_document_assets/135237114/N8KoQPEL2Sc3o4_PBGfzsB2DkKVqMt7OLFoLZJQHEaw-cove_K6LzhXq.png)

## Highlights


Script Gadget is an *existing* JS code on the page that may be used to bypass mitigations
> [View Highlight](https://read.readwise.io/read/01jh37trcjq01fknr9byrhsnxg)



Script Gadgets convert otherwise safe HTML tags and attributes into arbitrary JavaScript code execution.
> [View Highlight](https://read.readwise.io/read/01jh37v5dnnbgttndtb9zrvq6j)



• If a page with this gadget has an unfixed HTML injection, the attacker can inject data-text=”<script>” instead of injecting <script> • This lets the attacker bypass XSS mitigations that look for script.
 • Different gadgets bypass different mitigations
> [View Highlight](https://read.readwise.io/read/01jh37wkxv45vkrjvp9zx50app)



This HTML snippet: <div data-bind="value:'hello world'"></div>
> [View Highlight](https://read.readwise.io/read/01jh37zfs0g8edetrbm8fcq420)



These blocks create a gadget in Knockout that eval()s an attribute value. data-bind="value: foo" eval(“foo”) To XSS a Knockout-based JS application, attacker needs to inject: <div data-bind="value: alert(1)"></div>
> [View Highlight](https://read.readwise.io/read/01jh37zmt2vb58rw1pdntv2jz2)



Ajaxify gadget converts all <div>s with class=document-script into script elements. So if you have an XSS on a website that uses Ajaxify, you just have to inject: <div class="document-script">alert(1)</div>
> [View Highlight](https://read.readwise.io/read/01jh380mpyab6br7jrkxqsdybj)



Bootstrap has the "simplest" gadget, passing HTML attribute value into innerHTML.
 <div data-toggle=tooltip data-html=true title='<script>alert(1)</script>'> HTML sanitizers allow title attribute, because it’s usually safe.
 But they aren’t, when used together with Bootstrap and other data- attributes.
> [View Highlight](https://read.readwise.io/read/01jh381pngnwfkxsn0cm2cxv6v)



Require JS allows the user to specify the "main" module of a JavaScript file, and it is done through a custom data attribute, of which XSS filters and other mitigations aren't aware of.
 <script data-main='data:1,alert(1)' src='require.js'></script>
> [View Highlight](https://read.readwise.io/read/01jh3832zz9vwdbzver9knb4rj)



jQuery contains gadget that takes existing <script> tags, and reinserts them. We can inject a form and an input element to confuse the jQuery logic to reinsert our script: <form class="child"> <input name="ownerDocument"/><script>alert(1);</script></form> Strict-dynamic CSP blocks the <script>, but then jQuery reinserts it. Now it’s trusted and will execute.
> [View Highlight](https://read.readwise.io/read/01jh384cve27zvwd7n0e6jfgjy)



• We looked for Script Gadgets in 16 popular modern JS libraries. AngularJS 1.x, Aurelia, Bootstrap, Closure, Dojo Toolkit, Emberjs, Knockout, Polymer 1.x, Ractive, React, RequireJS, Underscore / Backbone, Vue.js, jQuery, jQuery Mobile, jQuery UI • It turned out they are prevalent in the above • Only one library did not have a a useful gadget • Gadgets we found were quite effective in bypassing XSS mitigations.
> [View Highlight](https://read.readwise.io/read/01jh388e0xe8kf28j3wayz3f1n)

