---
author: httptoolkit.com
aliases: [New Ways to Inject System CA Certificates in Android 14]
tags: [readwise/articles, android]
url: https://httptoolkit.com/blog/android-14-install-system-ca-certificate/
date: 2024-08-20
---
# New Ways to Inject System CA Certificates in Android 14

![rw-book-cover](https://httptoolkit.com/static/c226591ea7b920da3b29fe6a4a2cd00d/3c289/android-phone.jpg)

## Highlights


> In Android 14, system-trusted CA certificates will generally live in `/apex/com.android.conscrypt/cacerts`, and all of `/apex` is immutable.
> [View Highlight](https://read.readwise.io/read/01hbnvfwtka8yhxjx8zz22zhfm)



> That APEX cacerts path cannot be remounted as rewritable - remounts simply fail.
> [View Highlight](https://read.readwise.io/read/01hbnvj0v6cq5t13vpn0sntkc8)



> The `/apex` mount is [explicitly mounted](https://cs.android.com/android/platform/superproject/main/+/main:system/core/init/mount_namespace.cpp;l=97;drc=566c65239f1cf3fcb0d8745715e5ef1083d4bd3a) with PRIVATE propagation, so that all changes to mounts inside the `/apex` path are never shared between processes.
> [View Highlight](https://read.readwise.io/read/01hbnvnyjh1vztv4yjgtzerswt)



> using `nsenter`, we can run code in other namespaces!
> [View Highlight](https://read.readwise.io/read/01hbnvsjw2te3sagm788ndvz7e)



> • First, we need set up a writable directory somewhere. For easy compatibility with the existing approach, I'm doing this with a `tmpfs` mount over the (still present) non-APEX system cert directory:
>  mount -t tmpfs tmpfs /system/etc/security/cacerts
>  • Then you place the CA certificates you're interested in into this directory (e.g. you might want copy all the defaults out of the existing `/apex/com.android.conscrypt/cacerts/` CA certificates directory) and set permissions & SELinux labels appropriately.
>  • Then, use `nsenter` to enter the Zygote's mount namespace, and bind mount this directory over the APEX directory:
>  nsenter --mount=/proc/$ZYGOTE_PID/ns/mnt -- \
>  /bin/mount --bind /system/etc/security/cacerts /apex/com.android.conscrypt/cacerts
>  The Zygote process spawns each app, copying its mount namespace to do so, so this ensures all newly launched apps (everything started from now on) will use this.
>  • Then, use `nsenter` to enter each already running app's namespace, and do the same:
>  nsenter --mount=/proc/$APP_PID/ns/mnt -- \
>  /bin/mount --bind /system/etc/security/cacerts /apex/com.android.conscrypt/cacerts
>  Alternatively, if you don't mind the awkward UX, you should be able to do the bind mount on `init` itself (PID 1) and then run `stop && start` to soft-reboot the OS, recreating all the namespaces and propagating your changes everywhere (but personally I do mind the awkward reboot, so I'm ignoring that route entirely).
> [View Highlight](https://read.readwise.io/read/01hbnvvwstvzd0ve4qkbkbyear)



> second solution comes from [infosec.exchange/@g1a55er](https://infosec.exchange/@g1a55er/), who published [their own post](https://www.g1a55er.net/Android-14-Still-Allows-Modification-of-System-Certificates)
> [View Highlight](https://read.readwise.io/read/01hbnvxd7j5zs039k45sp1pk91)



> • You can remount `/apex` manually, removing the PRIVATE propagation and making it writable (ironically, it seems that entirely removing private propagation *does* propagate everywhere)
>  • You copy out the entire contents of `/apex/com.android.conscrypt` elsewhere
>  • Then you unmount `/apex/com.android.conscrypt` entirely - removing the read-only mount that immutably provides this module
>  • Then you copy the contents back, so it lives into the `/apex` mount directly, where it can be modified (you need to do this quickly, as [apparently](https://infosec.exchange/@g1a55er/111069489513139531) you can see crashes otherwise)
>  • This should take effect immediately, but they recommend killing `system_server` (restarting all apps) to get everything back into a consistent state
> [View Highlight](https://read.readwise.io/read/01hbnvycv347rjskdqvtqqfh8x)



> for both these solutions, this is a temporary injection - the certificates only last until the next reboot.
> [View Highlight](https://read.readwise.io/read/01hbnvytbnc37eeztvb8g2geby)

