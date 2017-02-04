---
title: Android setting up simple background service [tutorial]
layout: post
---

In order to start writing Service on Android first of all we need know about these things written below.

1. [Services](https://developer.android.com/guide/components/services.html) - what is services and how to code?
2. [AlarmManager](https://developer.android.com/reference/android/app/AlarmManager.html) - setting service up to run in certain time or periodically
3. [Broadcasts](https://developer.android.com/guide/components/broadcasts.html) - running service again when phone restarted

Let's start with Services.

## What is services and how to code?
A [Service](https://developer.android.com/reference/android/app/Service.html) is an application component that can perform long-running operations in the background, and it does not provide a user interface. Another application component can start a service, and it continues to run in the background even if the user switches to another application.

To create service we just need to create a class that extends Service.

```
public class YourService extends Service {

    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }
		
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        // your code here
        return START_NOT_STICKY;
    }
}
```

Returning **START_NOT_STICKY** means that if the process is killed with no remaining start command code then this service will be stopped, instead of restarting. It's actually usefull when you want to call **onStartCommand** when **AlarmManager** triggers in given time. You can learn more about result codes returned by function on this [link](https://android-developers.googleblog.com/2010/02/service-api-changes-starting-with.html).

After creating service include service to your **AndroidManifest.xml** under application tag.
```
<application
            android:allowBackup="true"
            android:icon="@mipmap/ic_launcher"
            android:label="@string/app_name"
            android:supportsRtl="true"
            android:theme="@style/AppTheme">
        <activity android:name=".MainActivity"
                  android:screenOrientation="portrait">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>

                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
        <service android:name=".MyService"/>
</application>
```

To launch service go to your **MainActivity** or another class where do you want to run this service.
```
	startService(new Intent(this, MyService.class));
```

## Setting service up to run in certain time or periodically

**AlarmManager** helps to set up time and run your service whenever you want.

In order to this you have to include **onDestroy** method to your **Service**. When your service will be stopped after **onStartCommand** **AlarmManager** will to start service in given time again.

```
public class YourService extends Service {

    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }
		
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        // your code here
	stopSelf(); // do write this line after your operations
        return START_NOT_STICKY;
    }
		
@Override
    public void onDestroy() {
//        super.onDestroy();
        Calendar calendar = Calendar.getInstance();
        calendar.setTimeInMillis(System.currentTimeMillis());
        calendar.set(Calendar.HOUR, 9); // running service everyday at 9
        AlarmManager alarm = (AlarmManager) getSystemService(ALARM_SERVICE);
        alarm.setRepeating(
                AlarmManager.RTC_WAKEUP,
                calendar.getTimeInMillis(),
                24*60*60*1000,
                PendingIntent.getService(this, 0, new Intent(this, MyService.class), 0)
        );
    }
}
```

## Running service again when phone restarted
First of all we need create a class that extends **BroadcastReceiver**. I'm calling my class AutoStart.

```
public class AutoStart extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        context.startService(new Intent(context, MyService.class));
    }
}
```

As you see in **onReceive** function we call to start our service that we wrote above.

After this procedures we need declare our class as a **receiver** and add **RECEIVE_BOOT_COMPLETED** permission to our **AndroidManifest.xml**

```
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
//add this before <application> tag
```

Adding our class as a receiver between **application** tag

```
<receiver android:name=".service.AutoStart">
  <intent-filter>
	  <action android:name="android.intent.action.BOOT_COMPLETED" />
  </intent-filter>
</receiver>
```

Clap! Clap!