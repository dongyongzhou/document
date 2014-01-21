---
layout: master
title: Android input event handling
---


## Overview

contents

- How does Android gets users` input event?
- How does Android handles the input event?
- How does Android send the event to the UI progress
- How does UI handles the event?

## Architecture

### Frameworks

#### WindowManagerService

WindowManagerService
->InputManager->nativeInit

#### InputManager


Framework/base/services/jni/com_android_server_InputManager.java

nativeInit==android_server_InputManager_nativeInit 

->NativeInputManager-> new EventHub/InputManager

InputManager类主要是负责管理input Event，有InputReader从EventHub读取事件，然后交给InputDispatcher进行分发

在InputManager中的initialize的初始化了两个线程。一个是inputReaderThread，负责从EventHub中读取事件，另外一个是InputDispatcherThread线程，主要负责分发读取的事件去处理

#### EventHub


EventHub就是Input子系统的HAL层了，负责将linux的所有的input设备打开并负责轮询读取他们的上报的数据

Framework/base/services/input/EventHub.cpp 

在eventHub初始化的时候直接进入到scanDevices Locked。而在内核里面所有的input device在注册的时候都会在linux的文件系统下的/dev/input 下面，代码中的while循环会对DEVICE_PATH(/dev/input)下的所有的设备节点调用openDeviceLocked 方法

首先通过open系统调用得到设备节点的文件描述符，然后新构造一个叫InputDeviceIdentifier类。接着通过对刚才得到的设备节点描述下ioctl的命令获取设备的一些简单信息，譬如：设备的名字，设备驱动的版本号，设备的唯一id，和描述符轮询的方式。得到的这些信息保存在InputDeviceIdentifier类里面。最后又构造了一个Device类，其中设备描述符和刚才的构造InputDeviceIdentifier类作为参数重新构造了Device类。然后在构造成功了Device类又会通过ioctl系统调用获取input设备的一些比较重要的参数。比如：设备上报事件的类型是相对事件还是绝对事件，相对事件一般是指像鼠标滑动，绝对事件就好比触摸屏上报的坐标，设备所属的class等一些比较重要的信息。举一些例子：

INPUT_DEVICE_CLASS_KEYBOARD(按键类型)，INPUT_DEVICE_CLASS_CURSOR(带游标类型：鼠标和轨迹球等)，INPUT_DEVICE_CLASS_ TOUCH(触摸类型：单点触摸或多点触摸)，INPUT_DEVICE_CLASS_TOUCH_MT(这个类型特指多 点触摸)等。如果一个设备的驱动没有指明设备的类型的话，那么他在
android中上报的数据时不会被处理的。这个函数的最后是将input设备的文件描述符加入到轮询的集合中去，如果接收到事件就会去处理。

#### InputDispatcher

dispatchOnce

#### InputReader

ReaderOnce

### Android2.2之前

- KeyQ： WmS内部类，继承于KeyInputQueue。创建KeyQ对象后，立即启动一个线程(对应对象)，不断调用native方法读取用户的UI操作消息，比如按键、触摸屏等，并放到消息队列QueueEvent中。
- InputDispatcherThread：也是WmS内部类。也会创建一个线程，不断从消息队列QueueEvent中取出用户消息，过滤后发送给当前活动的客户端程序。
- 应用窗口对象ViewRoot都包含一个W子类，是一个Binder类，InputDispatcherThread通过IPC方式调用W所提供的函数，从而发送消息给客户窗口。

### Android2.3之后

获取消息：使用C++实现，包括消息的加工

应用添加窗口->本地创建一个ViewRoot对象->通过IPC调用WmS中的Session对象的addWindow方法，从而请求WmS创建一个窗口->WmS将窗口信息保存在内部一个窗口列表InputMonitor->使用InputManager把窗口信息传递给InputDispatcher对象，并通过JNI传递到NativeInputManager

- InputReader线程C++：system_progress中，持续调用输入设备驱动读取用户输入的消息。
- InputDispatcherThread（C++）： 从消息队列中取出消息，根据NativeInputManager确定当前活动窗口。两种方式派发。
- 使用Linux PIPE机制传递消息对客户端。一是PIPE直接发送到客户窗口，例如触摸屏信息；另一是先派发到WMS中，由WMS经过一定的处理（例如系统按键消息HOME、电话按键等），如果WMS没有处理再派发到窗口。


## 代码及机制分析

- InputManager中的InputReader正在睡眠等待键盘事件的发生
- InputManager中的InputDispatcher正在等待InputReader从睡眠中醒过来并且唤醒它
- 应用程序也正在消息循环中等待InputDispatcher从睡眠中醒过来并且唤醒它。


A. 键盘事件发生，InputManager中的InputReader被唤醒，此前InputReader睡眠在/dev/input/event0这个设备文件上；

B. InputReader被唤醒后，它接着唤醒InputManager中的InputDispatcher，此前InputDispatcher睡眠在InputManager所运行的线程中的Looper对象里面的管道的读端上；

C. InputDispatcher被唤醒后，它接着唤醒应用程序的主线程来处理这个键盘事件，此前应用程序的主线程睡眠在Client端InputChannel中的前向管道的读端上；

D. 应用程序处理处理键盘事件之后，它接着唤醒InputDispatcher来执行善后工作，此前InputDispatcher睡眠在Server端InputChannel的反向管道的读端上，注意这里与第二个线索处的区别。


frameworks/base/services/input/

    Android.mk    InputApplication.cpp  InputDispatcher.h  InputManager.cpp  InputReader.h    PointerController.cpp  SpriteController.h
    EventHub.cpp  InputApplication.h    InputListener.cpp  InputManager.h    InputWindow.cpp  PointerController.h    tests
    EventHub.h    InputDispatcher.cpp   InputListener.h    InputReader.cpp   InputWindow.h    SpriteController.cpp

InputReader: threadLoop->loopOnce->processEventsLocked->processEventsForDeviceLocked->KeyboardInputMapper::process->processKey->notifyKey

InputDispatcher: notifyKey->enqueueInboundEventLocked->mInboundQueue.enqueueAtTail(entry);

Step 1. InputReader.pollOnce

Step 2. EventHub.getEvent

InputReaderThread线程会不民地循环调用InputReader.pollOnce函数来读入键盘事件，而实际的键盘事件读入操作是由EventHub.getEvent函数来进行的。如果当前没有键盘事件发生，InputReaderThread线程就会睡眠在EventHub.getEvent函数上，而当键盘事件发生后，就会把这个事件封装成一个RawEvent对象，然后返回到pollOnce函数中，执行process函数进一步处理

Step 3. InputReader.process

Step 4. InputReader.consumeEvent

首先从rawEvent中取得触发键盘事件设备对象device，然后调用它的process函数进行处理。

Step 5. InputDevice.process

这里的mMapper成员变量保存了一系列输入设备事件处理象，例如负责处理键盘事件的KeyboardKeyMapper对象、负责处理轨迹球事件的TrackballInputMapper对象以及负责处理触摸屏事件的TouchInputMapper对象， 它们是在InputReader类的成员函数createDevice中创建的。这里查询每一个InputMapper对象是否要对当前发生的事件进行处理。由于发生的是键盘事件，真正会对该事件进行处理的只有KeyboardKeyMapper对象。

Step 6. KeyboardInputMapper.process

这个函数首先会检查一下键盘扫描码是否正确，如果正确的话，就会调用processKey函数进一步处理。


Step 7. KeyboardInputMapper.processKey

这个函数首先对对按键作一些处理，例如，当某一个DPAD键被按下时，根据当时屏幕方向的不同，它所表示的意义也不同，因此，这里需要根据当时屏幕的方向来调整键盘码

最后，KeyboardInputMappger函数通知InputDispatcher，有键盘事件发生了：

Step 8. InputDispatcher.notifyKey
这个函数定义在InputDispatcher.cpp文件中：

最后，调用enqueueInboundEventLocked函数把这个按键事件封装成一个KeyEntry结构加入到InputDispatcher类的mInboundQueue队列中去

 从这个函数我们可以看出，在两种情况下，它的返回值为true，一是当加入该键盘事件到mInboundQueue之前，mInboundQueue为空，这表示InputDispatccherThread线程正在睡眠等待InputReaderThread线程的唤醒，因此，它返回true表示要唤醒InputDispatccherThread线程；二是加入该键盘事件到mInboundQueue之前，mInboundQueue不为空，但是此时用户按下的是Home键，按下Home键表示要切换App，我们知道，在切换App时，新的App会把它的键盘消息接收通道注册到InputDispatcher中去，并且会等待InputReader的唤醒，因此，在这种情况下，也需要返回true，表示要唤醒InputDispatccherThread线程。如果不是这两种情况，那么就说明InputDispatccherThread线程现在正在处理前面的键盘事件，不需要唤醒它。

回到前面的notifyKey函数中，根据enqueueInboundEventLocked函数的返回值来决定是否要唤醒InputDispatccherThread线程

Step 9. Looper.wake

它的作用就是用来唤醒睡眠在Looper对象内部的管道读端的线程，在我们的这个场景中，睡眠在Looper对象内部的管道读端的线程就是InputDispatccherThread线程了。

从上面InputManager启动过程的Step 15中，我们知道，此时InputDispatccherThread线程正在InputDispatcher类的dispatchOnceb函数中通过调用mLooper->loopOnce函数进入睡眠状态。当它被唤醒以后，它就会从InputDispatcher类的dispatchOnceb函数返回到InputDispatcherThread类的threadLoop函数，而InputDispatcherThread类的threadLoop函数是循环执行的，于是，它又会再次进入到InputDispatcher类的dispatchOnce函数来处理当前发生的键盘事件。

Step 10. InputDispatcher.dispatchOnce

它调用dispatchOnceInnerLocked函数来进一步处理这个键盘事件。

Step 11. InputDispatcher.dispatchOnceInnerLocked

首先，如果前面发生的键盘事件都已经处理完毕，那么这里的mPendingEvent就为NULL，又因为前面我们把刚刚发生的键盘事件加入了mInboundQueue队列，因此，这里mInboundQueue不为NULL，于是，这里就把mInboundQueue队列中的键盘事件取出来，放在mPendingEvent变量中：

由于这里发生的是键盘事件，即mPendingEvent->type的值为EventEntry::TYPE_KEY，于是，在接下来的switch语句中就会执行dispatchKeyLocked函数来分发键盘消息。

Step 12. InputDispatcher.dispatchKeyLocked

 InputDispatcher类中的mCurrentInputTargetsValid成员变量表示InputDispatcher是否已经标志出谁是当前激活的Activity窗口，如果没有，就需要通过findFocusedWindowTargetsLocked函数来把它找出来。当把当前激活的Activity窗口找出来以后，接下来就调用dispatchEventToCurrentInputTargetsLocked函数把键盘事件分发给它了。

Step 13. InputDispatcher.findFocusedWindowTargetsLocked

在Step 9中，当前处于激活状态的应用程序会通过调用InputDispatcher类setInputWindows函数把把当前获得焦点的Activity窗口设置到mFocusedWindow中去，因此，这里的mFocusedWindow不为NULL，于是，就通过了第一个if语句的检查。

第二个if语句检查权限问题，原来，这个键盘事件除了是由硬件触发的外，也可以由其它进程注入进来的，如果这个键盘事件是由其它进程注入进来的，那么entry->injectState就不为NULL，它里面包含了事件注册者的进程ID和用户ID，于是，这里就会调用checkInjectionPermission来检查这个事件注入者的进程ID和用户ID，看看它是否具有这个权限。这里我们不考虑这种情况，因此，这里的entry->injectState为NULL，于是，这个if语句的检查也通过了。

第三个if语句检查当前激活的Activity窗口是否是处于paused状态，如果是的话，也不用进一步处理了。一般情况下，当前激活的Activity窗口都是处于resumed状态的，于是，这个if语句的检查也通过了。

第四个if语句检查当前激活的Activity窗口是否还正在处理前一个键盘事件，如果是的话，那就要等待它处理完前一个键盘事件后再来处理新的键盘事件了。这里我们也假设当前激活的Activity窗口不是正在处理前面的键盘事件，因此，这个if语句的检查也通过了

最后，就调用addWindowTargetLocked函数把当前激活的Activity窗口添加到InputDispatcher类的mCurrentInputTargets成员变量中去。

 Step 14. InputDispatcher.addWindowTargetLocked

这个函数简单，就是把传进来的参数window添加到mCurrentInputTargets中去就完事了，后面InputDispatcher就会从mCurrentInputTargets中取出恰当的Activity窗口，然后把键盘事件分发给它。

回到Step 12中的dispatchKeyLocked函数，它接下来就调用dispatchEventToCurrentInputTargetsLocked来进一步处理了。

Step 15. InputDispatcher.dispatchEventToCurrentInputTargetsLocked

这个函数的实现也比较简单，前面我们已经把当前需要接受键盘事件的Activity窗口添加到mCurrentInputTargets中去了，因此，这里就分别把它们取出来，然后调用prepareDispatchCycleLocked函数把键盘事件分发给它们处理。

前面我们在分析应用程序注册键盘消息接收通道的过程时，在Step 18中（InputDispatcher.registerInputChannel），把Server端的InputChannel封装成了一个Connection，然后以这个InputChannel中的Receive Pipe Fd作为键值把这个Connection对象保存在mConnectionsByReceiveFd中。这里，既然我们已经通过mCurrentInputTargets得到了表示当前需要接收键盘事件的Activity窗口的InputTarget对象，而且这个InputTarget对象的inputChannel就表示当初在InputDispatcher中注册的Server端InputChannel，因此，这里就可以把这个Connection对象取出来，最后调用prepareDispatchCycleLocked函数来进一步处理。

Step 16. InputDispatcher.prepareDispatchCycleLocked

在开始处理键盘事件之前，这个函数会检查一下传进来的参数connection中的outboundQueue事件队列是否为空，如果不为空，就要看看当前要处理的事件和outboundQueue队列中的最后一个事件是不是同一个motion事件，如果是的话，并且从上面传进来的resumeWithAppendedMotionSample参数为true，这时候就要以流水线的方式来处理这些motion事件了。在我们这个情景中，要处理的是键盘事件，因此在上面Step 12中传进来的resumeWithAppendedMotionSample参数为false，因此，我们略过这种情况。
         
接下来，就会把当前的键盘事件封装成一个DispatchEntry对象，然后添加到connection对象的outboundQueue队列中去，表示当前键盘事件是一个待处理的键盘事件。    
         
当connection中的outboundQueue事件队列不为空，即wasEmpty为false时，说明当前这个Activity窗口正在处键盘事件了，因此，就不需要调用startDispatchCycleLocked来启动Activity窗口来处理这个事件了，因为一旦这个Activity窗口正在处键盘事件，它就会一直处理下去，直到它里的connection对象的outboundQueue为空为止。当connection中的outboundQueue事件队列为空时，就需要调用startDispatchCycleLocked来通知这个Activity窗口来执行键盘事件处理的流程了。

Step 17. InputDispatcher.startDispatchCycleLocked

这个函数主要围绕传进来的Connection对象做两件事情，

一是从它的outboundQueue队列中取出当前需要处理的键盘事件，然后把这个事件记录在它的内部对象inputPublisher中，
二是通过它的内部对象inputPublisher通知它所关联的Activity窗口，现在有键盘事件需要处理了。
第一件事情是通过调用它的InputPublisher对象的publishKeyEvent函数来完成的，
而第二件事情是通过调用它的InputPublisher对象的sendDispatchSignal来完成的。

Step 18. InputPublisher.publishKeyEvent

这个函数主要就是把键盘事件记录在InputPublisher类的成员变量mSharedMessage中了，这个mSharedMessage成员变量指向的是一个匿名共享内存。

Step 19. InputPublishe.sendDispatchSignal

这个函数很简单，它通过调用内部成员变量mChannel的sendSignal函数来通知相应的Activity窗口来处理键盘事件。

Step 20. InputChannel.sendSignal

这里所谓的发送信号通知，其实是通过向其内部一个管道的写端写入一个字符来实现的。前面我们分析应用程序注册键盘消息接收通道的过程时，在Step 21中（NativeInputQueue.registerInputChannel），它把一个InputChannel注册到应用程序主线程中的Looper对象中，然后应用程序的主线程就通过这个Looper对象睡眠等待在这个InputChannel中的前向管道中有新的内容可读了，这里的mSendPipeFd就是对应这个前向管道的写端。现在既然向这个前向管道的写端写入新的内容了，于是，应用程序的主线程就被唤醒了。

在前面分析应用程序注册键盘消息接收通道过程的Step 21中，我们也说过，当应用程序的主线程因为这个InputChannel中的前向管道的写端唤醒时，NativeInputQueue的成员函数handleReceiveCallback就会被回调，因此，接下来，应用程序的主线程就会被唤醒，然后执行NativeInputQueue的成员函数handleReceiveCallback。

Step 21. NativeInputQueue.handleReceiveCallback

这个函数首先是通过参数data获得当初注册InputChannel的NativeInputQueue对象，具体可以参考前面介绍的应用程序注册键盘消息接收通道过程的Step 21。接下来再通过参数receiveFd获得保存在这个NativeInputQueue对象中的mConnectionsByReceiveFd成员变量中的Connection对象。有了这个Connection对象后，就可以获得它内部的InputConsumer对象，这个InputConsumer对象是和上面的Step 18中介绍的InputPublisher对象相应的。
        
在InputChannel内部中，分别有一个InputPublisher对象和一个InputConsumer对象，对于Server端的InputChannel来说，它使用的是InputPublisher对象，通过它进行键盘消息的分发，而对于Client端的InputChannel来说，它使用的是InputConsumer对象，通过它进行键盘消息的读取。
        
获得了这个InputConsumer对象后首先是调用它的receiveDispatchSignal来确认是否是接收到了键盘消息的通知，如果是的话，再调用它的consume函数来把键盘事件读取出来，最后，调用Java层的回调对象InputQueue的DispatchKeyEvent来处理这个键盘事件。下面，我们就依次来分析这些过程。

 Step 22. InputConsumer.receiveDispatchSignal

这个函数很简单，它通过它内部对象mChannel来从前向管道的读端读入一个字符，看看是否是前面的Step 20中写入的INPUT_SIGNAL_DISPATCH字符。

Step 23. InputConsumer.consume

这个函数很简单，只要对照前面的Step 18（InputPublisher.publishKeyEvent）来逻辑来看就可以了，后者是往匿名共享内存中写入键盘事件，前者是从这个匿名共享内存中把这个键盘事件的内容读取出来。

回到Step 21中的handleReceiveCallback函数中，从InputConsumer中获得了键盘事件的内容（保存在本地变量inputEvent中）后，就开始要通知Java层的应用程序了。

Step 24. InputQueue.dispatchKeyEvent

 这个函数首先会创建一个FinishedCallback类型的对象finishedCallback，FinishedCallback是InputQueue的一个内部类，它继承于Runnable类。这个finishedCallback对象是提供给当前Activity窗口的，当它处理完毕键盘事件后，需要通过消息分发的方式来回调这个finishedCallback对象，以及InputQueue类处理一个手尾的工作，后面我们会分析到。

这里的inputHandler对象是在前面分析应用程序注册键盘消息接收通道的过程时，在Step 1（ViewRoot.setView）中传进来的：

它是ViewRoot类的一个成员变量mInputHandler。因此，这里将调用ViewRoot类的内部对象mInputHandler的成员函数handleKey来处理键盘事件。

Step 25. InputHandler.handleKey

这个函数定义在frameworks/base/core/java/android/view/ViewRoot.java文件中

这个函数首先调用其外部类ViewRoot的startInputEvent成员函数来把回调对象finishedCallback保存下来：

然后再调用其外部类ViewRoot的dispatchKey成员函数来进一步处这个键盘事件。

Step 26. ViewRoot.dispatchKey

ViewRoot不是直接处理这个键盘事件，而是把作为一个消息（DISPATCH_KEY）它放到消息队列中去处理，这个消息最后由ViewRoot类的deliverKeyEvent成员函数来处理。

 Step 27. ViewRoot.deliverKeyEvent

ViewRoot在把这个键盘事件分发给当前激活的Activity窗口处理之前，首先会调用InputMethodManager的dispatchKeyEvent成员函数来处理这个键盘事件。InputMethodManager处理完这个键盘事件后，再回调用这里的mInputMethodCallback对象的finishedEvent成员函数来把键盘事件分发给当前激活的Activity窗口处理。当然，在把这个键盘事件分发给InputMethodManager处理之前，ViewRoot也会先把这个键盘事件分发给当前激活的Activity窗口的dispatchKeyEventPreIme成员函数处理。

Step 28. InputMethodManager.dispatchKeyEvent

这个函数定义在frameworks/base/core/java/android/view/inputmethod/InputMethodManager.java文件中。这是一个输入法相关的类，我们这里就不关注了，只要知道当输入法处理完成之后，它就会调用ViewRoot类的mInputMehtodCallback对象的finishedEvent成员函数。

Step 29.  InputMethodCallack.finishedEvent

这个函数最终调用ViewRoot的dispatchFinishedEvent来进一步处理。

Step 30. ViewRoot.dispatchFinishedEvent

和前面的Step 26一样，ViewRoot不是直接处理这个键盘事件，而是把它作为一个消息（FINISHED_EVENT）放在消息队列中去，最后，这个消息由ViewRoot的handleFinishedEvent函数来处理。

Step 31. ViewRoot.handleFinishedEvent

如果InputMethodManager没有处理这个键盘事件，那么ViewRoot就会调用deliverKeyEventToViewHierarchy函数来把这个键盘事件分发给当前激活的Activity窗口来处理。

Step 32. ViewRoot.deliverKeyEventToViewHierarchy

这个函数首先会调用ViewRoot类的成员变量mView的dispatchKeyEvent来处理这个键盘事件，然后最调用ViewRoot类的finishInputEvent来处理手尾工作。

ViewRoot类的成员变量mView的类型为DecorView，它是由ActivityThread类第一次Resume当前的Activity窗口时创建的，具体可以参考ActivityThread类的handleResumeActivity成员函数，这里就不关注了。

Step 33. DecorView.dispatchKeyEvent

这里通过getCallback函数返回的是当前应用程序的激活的Activity窗口的Window.Callback接口，一般它不为NULL，因此，这个函数会调用Activity类的dispatchKeyEvent来处理这个键盘事件。

Step 34. Activity.dispatchKeyEvent

这里，Activity不是直接处理这个键盘事件，而是通过KeyEvent的dispatch转发一下。注意，KeyEvent的成中函数dispatch的第一个参数的类型是KeyEvent.Callback，而Activity实现了这个接口，因此，这里可以传this引用过去。

Step 35. KeyEvent.dispatch

 这里就根据一个键是按下（ACTION_DOWN）、还是松开（ACTION_UP）或者是一个相同的键被多次按下和松开（ACTION_MULTIPLE）等不同事件类型来分别调用Activity的onKeyDown、onKeyUp和onKeyMultiple函数了。

Activity窗口处理完这个键盘事件后，层层返回，最后回到Step 32中，调用finishInputEvent事件来处理一些手尾工，下面我们将会看到这些手尾工是什么。

Step 36. ViewRoot.finishInputEvent

ViewRoot类里面的成员变量mFinishedCallback是在前面Step 25中由InputQueue设置的，它是一个Runnable对象，实际类型是定义在InputQueue的内部类FinishedCallback，因此，这里调用它的run方法时，接下来就会调用InputQueue的内部类FinishedCallback的run成员函数：

 Step 37.  InputQueue.nativeFinished

这个函数只是简单只调用NativeInputQueue的finished方法来进一处处理。

Step 38. NativeInputQueue.finished

这个函数最重要的参数便是finishedToken了，通过它可以获得之前通知Java层的InputQueue类来处理键盘事件的Connection对象，它的值是在上面的Step 21（NativeInputQueue.handleReceiveCallback）中生成的：

它的实现很简单，只是把receiveFd（前向管道的读端文件描述符）、connectionId（Client端的InputChannel对应的Connection对象在NativeInputQueue中的索引）和messageSeqNum（键盘消息的序列号）三个数值通过移位的方式编码在一个jlong值里面，即编码在上面的finishedToken参数里面。

因此，在上面的finished函数里面，首先就是要对参数值finishedToken进行解码，把receiveFd、connectionId和messageSeqNum三个值分别取回来：

Step 39. InputConsumer.sendFinishedSignal

这个函数的实现很简单，只是调用其内部对象mChannel的sendSignal函数来执行发送信号的通知。前面我们已经说过，这里的mChannel的类型为InputChannel，它是注册在应用程序一侧的Client端InputChannel，它的成员函数sendSignal的定义我们在上面的Step 20中已经分析过了，这里不再详述，不过，这里和上面Step 20不一样的地方是，它里的通知方向是从反向管道的写端（在应用程序这一侧）到反向管道的读端（在InputDispatcher这一侧）。

前面我们在分析应用程序注册键盘消息接收通道的过程时，在Step 18（InputDispatcher.registerInputChannel）中，说到InputDispatcher把一个反向管道的读端文件描述符添加到WindowManagerService所运行的线程中的Looper对象中去，然后就会在这个反向管道的读端上睡眠等待有这个管道有新的内容可读。现在，InputConsumer往这个反向管道写入新的内容了，于是，InputDispatcher就被唤醒过来了，唤醒过来后，它所调用的函数是InputDispatcher.handleReceiveCallback函数，这与前面的Step 21的逻辑是一样的。

Step 40. InputDispatcher.handleReceiveCallack

这个函数首先是通过传进来的receiveFd参数（反向管道的读端文件描述符）的值取得相应的Connection对象：

Step 41. InputPublisher.receiverFinishedSignal

这里的逻辑和前面的Step 22中NativeInputQueue确认是否真的收到键盘事件分发的信号的逻辑是一致的，都是通过InputChannel的receiveSignal函数来确认是否在管道中收到了某一个约定的字符值，不过，这里约定的字符值为INPUT_SIGNAL_FINISHED。

回到前面的Step 40中，确认了是真的收到了键盘事件处理完成的信号后，就调用InputDispatcher的finishDispatchCycleLocked函数来执行一些善后工作了。

Step 42. InputDispatcher.finishDispatchCycleLocked

这个函数主要就是做了三件事情：

一是通知其它系统，InputDispatcher完成了一次键盘事件的处理：

二是调用相应的connection对象的内部对象inputPublisher来的reset函数来回收一些资源，它里面其实就是释放前面在Step 18（InputPublisher.publishKeyEvent）使用的匿名共享内存了：

三是调用InputDispatcher的startNextDispatchCycleLocked函数来处理下一个键盘事件：

因为正在处理当前这个键盘事件的时候，很有可能又同时发生了其它的键盘事件，因此，这里InputDispatcher还不能停下来，需要继续调用startNextDispatchCycleLocked继续处理键盘事件，不过下一个键盘事件的处理过程和我们现在分析的过程就是一样的了。

## 启动流程

WMS->WindowManagerService->InputManager->mInputManager.start();->nativeStart

InputManager.cpp->

    status_t result = mDispatcherThread->run("InputDispatcher", PRIORITY_URGENT_DISPLAY);
    result = mReaderThread->run("InputReader", PRIORITY_URGENT_DISPLAY);


## How does Android send the event to the UI progress

###应用窗口

main thread即ActivityThread，开始时就会进入一个Looper循环中，然后不断地从MessageQueue中读取消息。如果没有消息，就会进入Wait状态。

##Reference

- [Android应用程序键盘（Keyboard）消息处理机制分析（三）](http://blog.csdn.net/yuleslie/article/details/7079448)
- [Android Input子系统架构](http://wenku.baidu.com/link?url=HYz5l-L6X7F2SaKmx5NzFL8rtVJVCU7s8NC_XkeB7k7CkFKCnihlAtj_T8unM4hq6XhCXzk3Ycn2T1K_Feh6qiRIXEZqvskSb_0i7b-uqQ7)
- [安卓4.1: input系统从frameworks到kernel](http://blog.chinaunix.net/uid-27167114-id-3347185.html)
