Application written using **Kotlin** [^kotlin] can be recognized because they contain `.kt` files inside the APK archive as well as a `kotlin` folder. It supports both R8 and ProGuard for obfuscation and code shrinking [^kotlin-obf] . 

[^kotlin]: https://kotlinlang.org/
[^kotlin-obf]: https://developer.android.com/build/shrink-code

Eventually you can also find references to annotations like `@kotlin.Metadata`, `kotlin/Metadata`, etc.