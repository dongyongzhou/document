---
layout: master
title: Android logging system
---

# Android logging system

## System Archetecture 

## Kernel Driver 

## Reading and Writing Logs in Application

The Android logging system provides a mechanism for collecting and viewing system debug output. Logcat dumps a log of system messages, which include things such as stack traces when the emulator throws an error and messages that you have written from your application by using the Log class. You can run LogCat through ADB or from DDMS, which allows you to read the messages in real time.

Class 

> android.util.Log

API for sending log output.

- v(String, String) (verbose)
- d(String, String) (debug)
- i(String, String) (information)
- w(String, String) (warning)
- e(String, String) (error)

Generally, use the Log.v() Log.d() Log.i() Log.w() and Log.e() methods.

## Logcat 命令  

code: system/core/logcat/logcat.cpp

### use LogCat from within DDMS

For more information on how to use LogCat within DDMS, see Using DDMS.

### use LogCat on an ADB shell. 

 To run LogCat, through the ADB shell, the general usage is:

> [adb] logcat [<option>] ... [<filter-spec>] ...

options include:

-  -s              Set default filter to silent.
                  Like specifying filterspec '*:s'
-  -f <filename>   Log to file. Default to stdout
-  -r [<kbytes>]   Rotate log every kbytes. (16 if unspecified). Requires -f
-  -n <count>      Sets max number of rotated logs to <count>, default 4
-  -v <format>     Sets the log print format, where <format> is one of:

                  brief process tag thread raw time threadtime long

> brief — Display priority/tag and PID of originating process (the default format).

> process — Display PID only.

> tag — Display the priority/tag only.

> thread — Display process:thread and priority/tag only.

> raw — Display the raw log message, with no other metadata fields.

> time — Display the date, invocation time, priority/tag, and PID of the originating process.

> long — Display all metadata fields and separate messages with blank lines.


-  -c              clear (flush) the entire log and exit
-  -d              dump the log and then exit (don't block)
-  -t <count>      print only the most recent <count> lines (implies -d)
-  -g              get the size of the log's ring buffer and exit
-  -b <buffer>     Request alternate ring buffer, 'main', 'system', 'radio'
                  or 'events'. Multiple -b parameters are allowed and the
                  results are interleaved. The default is -b main -b system.

Viewing Alternative Log Buffers

> radio — View the buffer that contains radio/telephony related messages.

> events — View the buffer containing events-related messages.

> main — View the main log buffer (default)

-  -B              output the log in binary

filterspecs are a series of
  <tag>[:priority]

where <tag> is a log component tag (or * for all) and priority is:

-  V    Verbose
-  D    Debug
-  I    Info
-  W    Warn
-  E    Error
-  F    Fatal
-  S    Silent (supress all output)

'*' means '*:d' and <tag> by itself means <tag>:v

If not specified on the commandline, filterspec is set from ANDROID_LOG_TAGS.
If no filterspec is found, filter defaults to '*:I'

If not specified with -v, format is set from ANDROID_PRINTF_LOG
or defaults to "brief"


