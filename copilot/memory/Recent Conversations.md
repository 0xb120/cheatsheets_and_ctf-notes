## AIDL Services: Identification & Interaction
**Time:** 2026-01-25 19:11
**Summary:** AIDL (Android Interface Definition Language) services are identified by their `onBind()` method returning a `.Stub` binder and are defined using `.aidl` files. To interact with an AIDL service, one must reverse engineer the original `.aidl` file by examining the generated service interface code. Key indicators for reverse engineering include the `DESCRIPTOR` variable for the file path, methods throwing `RemoteException`, and `TRANSACTION_` integers for method order.
