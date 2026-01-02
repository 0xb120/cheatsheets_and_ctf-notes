---
author: Piotr Bazydlo (@chudyPB)
aliases:
  - XSS to RCE by Abusing Custom File Handlers - Kentico Xperience CMS
tags:
  - RW_inbox
  - readwise/articles
url: https://labs.watchtowr.com/xss-to-rce-by-abusing-custom-file-handlers-kentico-xperience-cms-cve-2025-2748/?__readwiseLocation=
date: 2025-05-12
---
# XSS to RCE by Abusing Custom File Handlers - Kentico Xperience CMS

![rw-book-cover](https://labs.watchtowr.com/content/images/2025/03/kentico-1.png)

## Highlights


We’re keen to walk through another vulnerability chain we put together in February - going from a Cross-Site Scripting (XSS) vulnerability to full Remote Code Execution on a target Kentico Xperience CMS install - before reporting to Kentico themselves for remediation. [](https://read.readwise.io/read/01jv26mtnbmdtdv08szyqks4m1)



There are two reasons why we decided to write about this chain today:
 1. The identified XSS is interesting from a technical perspective, relying on two fairly unusual (but minor) server-side flaws, which can be combined to achieve XSS.
 2. Within Kentico’s Xperience CMS, privileged users have access to extremely sensitive functionality (as per most CMSs). Theoretically, this functionality could be available to us via the XSS, and thus, we have a potential path to RCE. [](https://read.readwise.io/read/01jv26p51xz3r5ymp5gvn6f759)



Step 1 - Unauthenticated Resource Fetching Handler [](https://read.readwise.io/read/01jv26r4ns2y2gw4wcev5t452b)



While looking through the Kentico Xperience CMS codebase and mapping out unauthenticated functionality, we immediately stumbled into a handler that made us feel uneasy - taking file paths, from unauthenticated users, and returning the contents of said files. [](https://read.readwise.io/read/01jv26r9ncrt3x0jaa5v68rat9)



Kentico Xperience exposes the `CMS.UIControls.GetResourceHandler` handler through `/CMSPages/GetResource.ashx` endpoint.
 This handler is accessible without authentication, and its purpose is very similar to the resource handlers across all the software. [](https://read.readwise.io/read/01jv26rwmb4rknfabrze7tyegy)



Without going too deep, and as a brief tl;dr, these methods **won't allow you to traverse past the webroot of the application**. Simply - it means that we can potentially reach any file, but this file needs to be located within the webroot of the application. [](https://read.readwise.io/read/01jv26x4v92egy698ps715t9y9)



This is common .NET application behaviour, and to illustrate this we’ve walked through some examples. In the context of these examples, let’s assume that our webroot path is: `C:\\inetpub\\wwwroot\\Kentico13\\CMS`:
 • `image=../../../../../../../../../Windows/win.ini` - **Prohibited**, as we are reading a file outside of `C:\\inetpub\\wwwroot\\Kentico13` directory.
 • `image=wat.png` - **Allowed**, as we are reading a file that should reside inside of the webroot
 • `image=/App_Data/wat.png` - **Allowed**, as we are still inside the webroot. [](https://read.readwise.io/read/01jv26y82f58r3z128xk4zjny2)



we can summarize.
 1. `GetResourceHandler` allows you to read files from the Kentico webroot directory (and its child directories).
 2. You have several processors available: `file`, `image` and others.
 3. The code always verifies the file extension and the list of allowed extensions depends on the processor selected.
 Those who deal with application security likely noticed that `svg` is an allowed extension for image processing (also allowed in the `file` processor). [](https://read.readwise.io/read/01jv270q4bc2rsjjn68tve94je)



GET /CMSPages/GetResource.ashx?image=Iexist.svg HTTP/1.1 Host: hostname Connection: keep-alive [](https://read.readwise.io/read/01jv271yjwjva4mhw1brj0wv8w)



While we spotted this primitive fairly quickly when reviewing the Kentico code base, we assumed no unauthenticated attacker would be able to actually write an arbitrary SVG file to the webroot.
 Thus, we said “meh” and moved on with our lives. [](https://read.readwise.io/read/01jv272563r8k8e1r21rx1zx7b)



Step 2 - Temporary File Upload Primitive [](https://read.readwise.io/read/01jv27290m3kg9nbx92e7pc2de)



Life is brutal though. When you look for the missing pieces of a potential RCE vulnerability chain - the law of vuln research tells you that you will not succeed. If you don’t care and don’t bother, the same law of vuln research gives you everything you need. [](https://read.readwise.io/read/01jv272tvv25gy0keyc0kfneg7)



`CMS.DocumentEngine.Web.UI.ContentUploader` caught our attention.
 File upload possibilities are one of the most popular ways to achieve Remote Code Execution, and thus we were *forced* to investigate it. [](https://read.readwise.io/read/01jv273fw39f3hj9x8n87ncnsc)



this handler can be reached without any authentication through the `/CMSModules/Content/CMSPages/MultiFileUploader.ashx` endpoint. [](https://read.readwise.io/read/01jv273r2jhtzq062h2zm7fhbp)



`ContentUploader` is used to upload files (surprise), but there are strong checks on the extension types once again. This includes a whitelist-like check that contains the following extensions by default:
 pdf, doc, docx, ppt, pptx, xls, xlsx, xml, bmp, gif, jpg, jpeg, png, wav, 
 mp3, mp4, mpg, mpeg, mov, avi, rar, zip, txt, rtf, webp [](https://read.readwise.io/read/01jv274t8qkrvxah1dq7ecmmzm)



We can reach the handler with the following sample HTTP Request:
 POST /KCMSModules/Content/CMSPages/MultiFileUploader.ashx HTTP/1.1
 Host: hostname
 Content-Length: X
 Content-Type: application/octet-stream
 content [](https://read.readwise.io/read/01jv275rbvxv9770dwp1a6ga30)



> Side note (and extremely important note): If you don’t provide the `InstanceGuid`, it will default to a GUID consisting of zeros only
 Let’s take stock - at this point, we appear to be able to:
 • Write files of a specific subset of permitted file extensions
 • To a directory that begins with a temporary directory specified in the Kentico codebase.
 Given this is a temporary file upload function though, it’s natural to be concerned that files uploaded here are removed instantly after they have been processed. [](https://read.readwise.io/read/01jv279798z2qkmmc70k8ts9gk)



if `uploadHelper.Complete` is `false`, we never reach the file removal method. [](https://read.readwise.io/read/01jv27b2k59n4np34mw3t6ha4x)



to illustrate this - to upload a temporary file, we can execute the following sample HTTP Request:
 POST /CMSModules/Content/CMSPages/MultiFileUploader.ashx?Filename=myfile.txt&Complete=false HTTP/1.1
 Host: hostname
 Content-Length: 6
 Content-Type: application/octet-stream
 myfile
 As a result, we have the file uploaded to the `~\\App_Data\\CMSTemp\\MultiFileUploader\\00\\00000000-0000-0000-0000-000000000000` path [](https://read.readwise.io/read/01jv27bbv1zxng2zncnsrndk3y)



You likely noticed that the GUID used in the file path is entirely 0’s - this is because, as mentioned above, if we don’t provide InstanceGuid, 0’s are defaulted to putting our file in an entirely predictable location. [](https://read.readwise.io/read/01jv27bpf9f2xh10xs6exd53xp)



Step 3 - Custom File Handler [](https://read.readwise.io/read/01jv27cj4qmb5ayz3xsk5a6scv)



Life goes on, and research continues. It is fairly common to find ‘bugs’ that aren’t quite ‘vulnerabilities’ and that’s life.
 Given a bit more thought, a line between the bugs suddenly appeared in our constrained minds and we realized that there is a possibility we might be further along in solving this than we thought. [](https://read.readwise.io/read/01jv27d8ws3ytds4z6fr29jsrm)



Back to the `GetResourceHandler` we go!
 At the end of step 1, we showed you that the file contents are being ultimately retrieved with this method:
 result = CMS.IO.File.ReadAllBytes(path); [](https://read.readwise.io/read/01jv27f3ydqg88wja652mg8e9a)



this is not the regular .NET `File.ReadAllBytes` method!
 It is, of course, a custom wrapper implemented by Kentico: `CMS.IO.File.ReadAllBytes`. There are several things happening in it, but you need to know only one of them. [](https://read.readwise.io/read/01jv27fdf1g2sk7gf2mmcz0b9t)



This method internally tries to retrieve something called `StorageProvider`. Among a few things, this provider defines how file reads are handled/performed. [](https://read.readwise.io/read/01jv27fp38vq27spc5vx1ve9fb)



Do you remember that our temporary file upload primitive allows us to upload ZIP files? Does this mean that we can upload a ZIP file to the temporary location provided by the upload handled, to our predictable location, and then try to read it with the custom `ZipProvider` provided by Kentico? [](https://read.readwise.io/read/01jv27gh0cnv8r9412vck021rn)



Let’s assume that we provide the following path:
 `/some/path/to/[poc.zip]/poc.svg`
 Kentico’s custom file handler has logic to read ZIP files for us in memory, and thus will perform the following operations:
 • It will read the ZIP file from the `/some/path/to/poc.zip` path into memory.
 • Then, it will retrieve the `poc.svg` file from this ZIP file! [](https://read.readwise.io/read/01jv27h8xkjjexdpfdb1gx4es3)



We can abuse this to, from an unauthenticated perspective, upload a ZIP file that contains an SVG file, and read it!
 The entire attack scenario is as follows:
 • Create a malicious `poc.svg` file.
 • Create a `poc.zip`, which stores `poc.svg`.
 • Upload `poc.zip` to a temporary location.
 • Use the resource handler to read the `~/temp/location/[poc.zip]/poc.svg`.
 • The file extension is `svg` , which is allowed.
 • MIME type is set on the basis of the `svg` extension.
 • XSS exploited! [](https://read.readwise.io/read/01jv27hwhn093kfg5mvv61b4mz)

