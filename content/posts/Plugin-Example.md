+++
date = '2025-04-20T17:32:39+05:30'
draft = true
title = 'Android Plugin App Example'
+++

#### Why ?
> Assume having an App with some functions that can be customized by the user 
The App already has some, but users want **more** :| 
This is a very very basic example on how that can be done


### Idea
```
1. Plugin App (where the customizations lives) have a service with a custom action (ex - com.example.contexttheme.PLUGIN )
2. Use PackageManager to get the Apps with custom action
3. Finally Using ClassLoaders to get classess and invoking methods on them to get results
```

#### Getting Plugins with required action
```kt{linenos=inline}
val pm = this.packageManager
val list = pm.queryIntentServices(Intent("com.example.contexttheme.PLUGIN"), 0)
val textViewIds = arrayOf(R.id.plugin_text1, R.id.plugin_text2)
Log.i("gggg", "size = ${list.size}");

for (i in list.indices) {
    try {
        Log.i("gggg", "li serviceInfo = " + list[i].serviceInfo.name);
        val appInfo = pm.getApplicationInfo(list[i].serviceInfo.packageName, 0)
        val apkPath = appInfo.sourceDir
        Log.i("gggg", "path  = ${apkPath}")
        val dir = getDir("dex_opt", Context.MODE_PRIVATE)
        val classLoader = PathClassLoader(apkPath, javaClass.classLoader)
        
        val pluginClass =
            classLoader.loadClass("com.example.plugin${i}.PluginTextProvider")
        val pluginInstance = pluginClass.newInstance()

        val textMethod = pluginClass.getMethod("getText")
        val text: String = textMethod.invoke(pluginInstance).toString()
        Log.i("gggg", "text for = ${text}")
        val textView: TextView = findViewById(textViewIds[i])
        textView.text = text
    } catch (e: Exception) {
        Log.e("gggg", "Print exception  ${e.message}")
        e.printStackTrace()
    }
}
```
---



#### Main App Manifest
```kt{linenos=inline}
<queries>
    <intent>
        <action android:name="com.example.contexttheme.PLUGIN" />
    </intent>
</queries>
```
---


#### Plugin App Manifest
```kt{linenos=inline}
<service
    android:name=".MyService"
    android:exported="true">
        <intent-filter>
            <action android:name="com.example.contexttheme.PLUGIN"></action>
        </intent-filter>
</service>
```
---


#### Custom classes implemented in Plugin APKs
```kt{linenos=inline}
// Plugin2 Apk
class PluginTextProvider {
    public fun getText() = "Hello from plugin2"
}

// Plugin1 Apk
class PluginTextProvider {
    public fun getText() = "Hello from plugin1"
}

```
---




