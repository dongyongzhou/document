---
layout: master
title: Excessive JNI References
---

## Overview


* Android Watchdog fires
* Fatal Exception in System Process
* Native Crash in System Process
* Excessive JNI global references

## Android Watchdog fires

## Fatal Exception in System Process

./base/core/java/com/android/internal/os/RuntimeInit.java

    private static class UncaughtHandler implements Thread.UncaughtExceptionHandler {
        public void uncaughtException(Thread t, Throwable e) {
            try {
                // Don't re-enter -- avoid infinite loops if crash-reporting crashes.
                if (mCrashing) return;
                mCrashing = true;

                if (mApplicationObject == null) {
                    Slog.e(TAG, "*** FATAL EXCEPTION IN SYSTEM PROCESS: " + t.getName(), e);
                } else {
                    Slog.e(TAG, "FATAL EXCEPTION: " + t.getName(), e);
                }


## Excessive JNI global references

Ordinarily, objects are discarded when nothing else in the virtual
heap holds a reference to them.  This is a problem for native code,
which can be holding an object reference that isn't visible to the
garbage collector.  JNI allows native code to convert a reference into
a "global" reference, which the garbage collector can see.

To help find memory leaks, a limit is placed on the total number of
global references available to the VM.  (This should only be enabled
on the emulator.) 


### Where is the detecting

./dalvik/vm/Jni.cpp:                    LOGW("Excessive JNI global references (%d)", count);

./dalvik/vm/Jni.cpp:                    LOGE("Excessive JNI global references (%d)", count);

       /* watch for "excessive" use; not generally appropriate */
            if (count >= gDvm.jniGrefLimit) {
                JavaVMExt* vm = (JavaVMExt*) gDvm.vmList;
                if (vm->warnError) {
                    dvmDumpReferenceTable(&gDvm.jniGlobalRefTable,"JNI global");
                    LOGE("Excessive JNI global references (%d)\n", count);
                    dvmAbort();
                } else {
                    LOGW("Excessive JNI global references (%d)\n", count);
                }
            }

### Where is the detecting
其中：上限由

./dalvik/vm/Init.cpp

        } else if (strncmp(argv[i], "-Xjnigreflimit:", 15) == 0) {
            int lim = atoi(argv[i] + 15);
            if (lim < 200 || (lim % 100) != 0) {
                dvmFprintf(stderr, "Bad value for -Xjnigreflimit: '%s'\n",
                    argv[i]+15);
                return -1;
            }
            gDvm.jniGrefLimit = lim;


Xjnigreflimit由设置：


./base/core/jni/AndroidRuntime.cpp:int AndroidRuntime::startVm(JavaVM** pJavaVM, JNIEnv** pEnv)

int AndroidRuntime::startVm(JavaVM** pJavaVM, JNIEnv** pEnv)
{

    LOGD("CheckJNI is %s\n", checkJni ? "ON" : "OFF");
    if (checkJni) {
        /* extended JNI checking */
        opt.optionString = "-Xcheck:jni";
        mOptions.add(opt);

        /* set a cap on JNI global references */
        opt.optionString = "-Xjnigreflimit:2000";
        mOptions.add(opt);

        /* with -Xcheck:jni, this provides a JNI function call trace */
        //opt.optionString = "-verbose:jni";
        //mOptions.add(opt);
    }

默认为2000



/dalvik/vm/Init.cpp

dvmAbort();->abort()

     * If we call abort(), all threads in the process receives a SIGABRT.
     * debuggerd dumps the stack trace of the main thread, whether or not
     * that was the thread that failed.
