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
