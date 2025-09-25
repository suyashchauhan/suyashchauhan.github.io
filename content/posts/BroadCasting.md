+++
date = '2025-04-28T22:27:10+05:30'
draft = true
title = 'BroadCasting'
+++



Main App  
  - Define Reciever
```kt{linenos=inline}
class MyReciever : BroadcastReceiver() {
    override fun onReceive(context: Context?, intent: Intent?) {
        Log.e("gggg", "Broadcast recieved");
        intent?.let {
            val data = it.getIntExtra("myVal", 0)
            val data2 = it.getStringExtra("asdf")
            Log.e("gggg", "extraData = $data $data2 ${it.extras}");
        }
    }
}
```

 - Register Reciever
```kt{linenos=inline}
// Register
ContextCompat.registerReceiver(
    this,
    myReciever,
    IntentFilter("com.example.contexttheme.ACTION_UPDATE"),
    ContextCompat.RECEIVER_EXPORTED
)

```
 - Unregister Reciever
```kt{linenos=inline}
// Unregister
this.unregisterReceiver(myReciever);
```
---

In a secondary App 
 - Fire Intent

```kt{linenos=inline}
val button = findViewById<Button>(R.id.broadcast_button)
button.setOnClickListener {
    val intentToBeFired = Intent("com.example.contexttheme.ACTION_UPDATE")
    val str: String = "MeaningFull Data"
    intentToBeFired.putExtra("your_string", str)
    intentToBeFired.putExtra("your_int",7);
    sendBroadcast(intentToBeFired)
}
```
---

### Discussion Points

**Q: How to protect any other app to handle a broadcast that my App has sent ?**  
Send Broadcast with permission , any other app which wanted to recieve should also have that permission declared in manifest

Main App
```kt{linenos=inline}
// Define permission and use it
<permission
        android:name="com.example.contexttheme.SECURE_BROADCAST_PERMISSION" />
<uses-permission android:name="com.example.contexttheme.SECURE_BROADCAST_PERMISSION" />

// Register reciever using permission
registerReceiver(
            myReciever,
            IntentFilter("com.example.contexttheme.ACTION_UPDATE"),
            "com.example.contexttheme.SECURE_BROADCAST_PERMISSION",
            null,
            RECEIVER_EXPORTED
        )


```
secondary App

```kt{linenos=inline}
// Declare permission to be used
<uses-permission android:name="com.example.contexttheme.SECURE_BROADCAST_PERMISSION" />

// send Broadcast with that permission
sendBroadcast(intentToBeFired, "com.example.contexttheme.SECURE_BROADCAST_PERMISSION")
```

