---
layout: master
title: Crash
---

# Crash Handler

## Reference


## Android Bad Behavior

Reference: development/apps/Development/src/com/android/development/BadBehaviorActivity.java

### Crash the main app thread

`throw new BadBehaviorException()

 private static class BadBehaviorException extends RuntimeException {

        BadBehaviorException() {
            super("Whatcha gonna do, whatcha gonna do",
                    new IllegalStateException("When they come for you"));
        }
    }`


### Crash an auxiliary app thread

new Thread() {
                    @Override
                    public void run() { throw new BadBehaviorException(); }
                }.start();


### Crash the native process

// For some reason, the JVM needs two of these to get the hint
                Log.i(TAG, "Native crash pressed -- about to kill -11 self");
                Process.sendSignal(Process.myPid(), 11);
                Process.sendSignal(Process.myPid(), 11);
                Log.i(TAG, "Finished kill -11, should be dead or dying");


### Crash the system server

     try {
                    IBinder b = ServiceManager.getService(POWER_SERVICE);
                    IPowerManager pm = IPowerManager.Stub.asInterface(b);
                    pm.crash("Crashed by BadBehaviorActivity");
                } catch (RemoteException e) {
                    Log.e(TAG, "Can't call IPowerManager.crash()", e);
     }


W/dalvikvm(15213): threadid=63: thread exiting with uncaught exception (group=0x40a859d8)
E/AndroidRuntime(15213): *** FATAL EXCEPTION IN SYSTEM PROCESS: PowerManagerService.crash()
E/AndroidRuntime(15213): java.lang.RuntimeException: Crashed by BadBehaviorActivity
E/AndroidRuntime(15213):        at com.android.server.PowerManagerService$12.run(PowerManagerService.java:2621)
W/AudioFlinger(15183): power manager service died !!!
W/Sensors (15413): sensorservice died [0x20fd990]
I/ActivityThread(15413): Removing dead content provider: settings
I/ActivityThread(16353): Removing dead content provider: settings
I/ActivityThread(15447): Removing dead content provider: settings
I/ActivityThread(15742): Removing dead content provider: settings
I/ActivityThread(15487): Removing dead content provider: settings
I/ActivityThread(15395): Removing dead content provider: settings
I/ActivityThread(15303): Removing dead content provider: settings
D/memalloc(  103): /dev/pmem: Freeing buffer base:0x41723000 size:20480 offset:1843200 fd:30
D/memalloc(  103): /dev/pmem: Freeing buffer base:0x418ea000 size:614400 offset:3706880 fd:51
D/memalloc(  103): /dev/pmem: Freeing buffer base:0x41bd8000 size:614400 offset:6778880 fd:66
D/memalloc(  103): /dev/pmem: Freeing buffer base:0x41566000 size:614400 offset:20480 fd:45
D/memalloc(  103): /dev/pmem: Freeing buffer base:0x41aac000 size:614400 offset:5550080 fd:36
E/BadBehaviorActivity(16353): Can't call IPowerManager.crash()
E/BadBehaviorActivity(16353): android.os.DeadObjectException
E/BadBehaviorActivity(16353):   at android.os.BinderProxy.transact(Native Method)
E/BadBehaviorActivity(16353):   at android.os.IPowerManager$Stub$Proxy.crash(IPowerManager.java:524)
E/BadBehaviorActivity(16353):   at com.android.development.BadBehaviorActivity$1.onClick(BadBehaviorActivity.java:138)
E/BadBehaviorActivity(16353):   at android.view.View.performClick(View.java:3511)
E/BadBehaviorActivity(16353):   at android.view.View$PerformClick.run(View.java:14105)
E/BadBehaviorActivity(16353):   at android.os.Handler.handleCallback(Handler.java:605)
E/BadBehaviorActivity(16353):   at android.os.Handler.dispatchMessage(Handler.java:92)
E/BadBehaviorActivity(16353):   at android.os.Looper.loop(Looper.java:137)
E/BadBehaviorActivity(16353):   at android.app.ActivityThread.main(ActivityThread.java:4424)
E/BadBehaviorActivity(16353):   at java.lang.reflect.Method.invokeNative(Native Method)
E/BadBehaviorActivity(16353):   at java.lang.reflect.Method.invoke(Method.java:511)
E/BadBehaviorActivity(16353):   at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:787)
E/BadBehaviorActivity(16353):   at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:554)
E/BadBehaviorActivity(16353):   at dalvik.system.NativeStart.main(Native Method)
I/ActivityThread(16140): Removing dead content provider: settings
I/ActivityThread(15976): Removing dead content provider: settings
I/ActivityThread(15571): Removing dead content provider: settings
E/SurfaceTexture(  103): [com.android.development/com.android.development.BadBehaviorActivity] dequeueBuffer: SurfaceTexture has been abandoned!

### Report a WTF condition

What a Terrible Failure: Report a condition that should never happen. The error will always be logged at level ASSERT with the call stack. Depending on system configuration, a report may be added to the android.os.DropBoxManager and/or the process may be terminated immediately with an error dialog.



### ANR (Stop responding for 20 seconds)

                Log.i(TAG, "ANR pressed -- about to hang");
                try { Thread.sleep(20000); } catch (InterruptedException e) { Log.wtf(TAG, e); }
                Log.i(TAG, "hang finished -- returning");

### ANR starting an Activity

                Log.i(TAG, "ANR pressed -- about to hang");
                try { Thread.sleep(20000); } catch (InterruptedException e) { Log.wtf(TAG, e); }
                Log.i(TAG, "hang finished -- returning");

### ANR receiving a broadcast Intent

                sendOrderedBroadcast(new Intent("com.android.development.BAD_BEHAVIOR"), null);

### ANR starting a Service

                startService(new Intent(BadBehaviorActivity.this, BadService.class));


### System ANR (in ActivityManager)

Intent intent = new Intent(BadBehaviorActivity.this, BadBehaviorActivity.class);
                Log.i(TAG, "ANR system pressed -- about to engage");
                try {
                    ActivityManagerNative.getDefault().setActivityController(
                        new BadController(20000));
                } catch (RemoteException e) {
                    Log.e(TAG, "Can't call IActivityManager.setActivityController", e);
                }
                startActivity(intent.putExtra("dummy", true));


### Wedge system (5 minute system ANR)

 Intent intent = new Intent(BadBehaviorActivity.this, BadBehaviorActivity.class);
                Log.i(TAG, "Wedge system pressed -- about to engage");
                try {
                    ActivityManagerNative.getDefault().setActivityController(
                        new BadController(300000));
                } catch (RemoteException e) {
                    Log.e(TAG, "Can't call IActivityManager.setActivityController", e);
                }
                startActivity(intent.putExtra("dummy", true));

## ActivityManager

An activity’s state is managed by the runtime’s ActivityManager
