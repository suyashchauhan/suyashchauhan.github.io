+++
date = '2025-04-20T22:37:14+05:30'
draft = true
title = 'Bound Service AIDL'
+++




#### Why or UseCases?
> 
For your application you need to connect to a download service which is responsible
for downloading the files and on complete inform you about the completion



<!-- ### Idea
```

``` -->

{{< details summary="Steps" >}}
- [**Setting up AIDL Interfaces for IPC**](#setting-up-AIDL-folder-with-files)
- **Setting Up service**
    - [Implement Binder by implementing registerCallback and unregister callback](#main-service)
    - Use RemoteCallbackList to maintain a list of - connections to our service
    - sending Important updates to client by invoking callback
- **Setting Up Client**
    - [Important Permission for Accessing Service](#app-manifest-client)
    - [Setting up callback and define actions to perform](#callback-client)
    - [setting up connection object and registering callback](#connection-object-client)

{{< /details >}}



#### Setting Up AIDL Folder with files
![alt text](/images/image.png)
```kt{linenos=inline}
interface IMyAidlInterface {
    int getPid();
    void registerCallback(IRemoteServiceCallback cb);
    void unregisterCallback(IRemoteServiceCallback cb);
}

interface IRemoteServiceCallback {  
    void valueChanged(int newValue);
}
```
---

#### App Manifest (Service)
```kt{linenos=inline}
<service
    android:name=".MyService"
    android:exported="true">
        <intent-filter>
            <action android:name="com.example.contexttheme.PLUGIN"></action>
        </intent-filter>
</service>
```

#### Main Service 
```kt{linenos=inline }
class MyService : Service() {

    val callbackList = RemoteCallbackList<IRemoteServiceCallback>()
    // RemoteCallbackList maintains a list of active connections to clients

    override fun onCreate() {
        super.onCreate()
        Log.i("gggg", "Creating Service")
    }

    private fun updateObservers() {
        val n = callbackList.beginBroadcast()
        for (i in 0 until n) {
            try {
                callbackList.getBroadcastItem(i).valueChanged(i);
            } catch (e: RemoteException) {
                Log.i("gggg", "Remote exception occurs on sending messages")
            }
        }
    }

    override fun onBind(intent: Intent?): IBinder? {
        Log.i("gggg", "Binding ... ");
        return binder
    }

    override fun onUnbind(intent: Intent?): Boolean {
        Log.i("gggg", "UnBinding ... ");
        return super.onUnbind(intent)
    }

    override fun onDestroy() {
        super.onDestroy()
        Log.i("gggg", "onDestroy... ");
        updateObservers()
        callbackList.kill()
    }

    private val binder = object : IMyAidlInterface.Stub() {
        override fun getPid(): Int {
            updateObservers()
            return Process.myPid()
        }

        override fun registerCallback(cb: IRemoteServiceCallback) {
            Log.i("gggg", "registering Callback")
            callbackList.register(cb)
        }

        override fun unregisterCallback(cb: IRemoteServiceCallback?) {
            Log.i("gggg", "unregister Callback")
            callbackList.unregister(cb)
        }
    }

}
```
---



#### App Manifest (Client)
```kt{linenos=inline}
<queries>
        <package android:name="com.example.contexttheme" />
</queries>
```
##### In order to ***explicitly*** fire an intent when binding 
---


#### Connecting to Service (Client)
```kt{linenos=inline}
val intent = Intent(action)
intent.setPackage("com.example.contexttheme");
this.bindService(intent, mConnection, Context.BIND_AUTO_CREATE)
```


#### Connection Object (Client)
```kt{linenos=inline}
    val action = "com.example.contexttheme.service";
    private val mConnection = object : ServiceConnection {

        // Called when the connection with the service is established.
        override fun onServiceConnected(className: ComponentName, service: IBinder) {
            myservice = IMyAidlInterface.Stub.asInterface(service)
            myservice?.registerCallback(mCallback)
            bound = true
        }

        // Called when the connection with the service disconnects unexpectedly.
        override fun onServiceDisconnected(className: ComponentName) {
            Log.e("gggg", "Service has unexpectedly disconnected")
            myservice = null
            bound = false
        }
    }

 
```


#### Callback (Client)
```kt{linenos=inline}
private val mCallback: IRemoteServiceCallback = object : IRemoteServiceCallback.Stub() {
    override fun valueChanged(newValue: Int) {
        Log.i("gggg", "value has been changed")
    }
}
```
---
   

