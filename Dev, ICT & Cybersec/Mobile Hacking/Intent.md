
[Intents](https://developer.android.com/reference/android/content/Intent) are the key of **internal communication** between the elements that make up an application. They are abstract descriptions containing information about the operation to be performed.
![|800](../../zzz_res/attachments/intents.png)

Two forms of intents exist:
- **Explicit**: they have specified a component that determines which exact class should be executed. In most cases they are intents that have the sole task of starting the various internal activities of the application, without interacting with external applications.
- **Implicit**: did not specify a component, but instead must include enough information for the system to determine which of the available components is best to use.