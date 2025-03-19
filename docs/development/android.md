# Android Development Guide

The following page is a collection of helpful and useful tricks for Android App Development and general Android use. This can include apps, tools, and how-tos when developing certain android components

==Design==
* Build with Asynchronous in Mind
* Modularize as much as possible
** Android likes to create heavy cupling with the UI which is not maintainably desirable
** Sometimes heavy coupling is unavoidable
* Interfaces and callbacks are primary method of communication between components - unless you want to use Broadcasts or Messages
** Service <-> Activity
** Activity <-> Fragment
* Spaghetti Code Is A Serious Problem

==Development==

General Tips:
* Use try/catch a fair amount. Errors show up in odd places
* Use isFinishing in activities, to ensure UI changes don't occur when the Activity is being deconstructed
* finish() and startActivity() are asynchronous - onDelete() is not gauranteed to be called or completed before onCreate() for the new activity is run

===Services===
Multiple simultanious services can largely cause a mess, and is not a recommended approach when developing applications.

When creating services using the `bindService` and `unbindService` calls - always call then in context of the `applicationContext`. Thus making these calls as such

<syntaxhighlight lang="java" line="line">
//Kotlin code
applicationContext.bindService()
applicationContext.unbindService()
</syntaxhighlight>



This is because binding events take the current context when determinging what services need to be started or stopped (and used to check if its leaking). With multiple bind and unbind calls you may end up
with some bind soup where an unbind won't unbind because the context which the bind call was made in was different. Using the applicationContext you can avoid this problem

===Broadcast Receivers=== 
If registering a service to receive broadcasts, its best to make the register call in the Service's `onCreate` call and possibly again in the `onStartCommand` call. Calling `registerReceiver`
using the `applicationContext` is also recommended so your calls should look like this
 
<syntaxhighlight lang="java" line="line">
//Kotlin code
applicationContext.registerReceiver()
applicationContext.unregisterReceiver()
</syntaxhighlight>

This is again to avoid the context soup problem described when binding and unbinding services
