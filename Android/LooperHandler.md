### Handler, Looper, Message Queue, Runnable源码分析

大家做Android开发难免不和这几个哥们打交道。而且基本上，面试必问。为了学习，也为了搞清楚这个东西，我决定从源码的角度分析这几个哥们。

首先说一下关系：

- `Runnable`和`Message`可以放入某个`MessageQueue`里面，形成一个集合
- `Looper`循环的从`Message Queue`里面取出一个东西，交由`Handler`进行处理
- `Handler`才是真正干活的地方





既然`Handler`才是干活的，我们来先看看这个哥们长啥样子。

源码代码位置：`$ANDROID_BASE/frameworks/base/core/android/os/Handler.java`

`$ANDROID_BASE/frameworks/base/core/android/os/Looper.java`

`$ANDROID_BASE/frameworks/base/core/android/os/MessageQueue.java`

`Handler.java`：

```java
public class Handler {
    ...
    
    final Looper mLooper;
    final MessageQueue mQueue;
    final Callback mCallback;
}
```

`Looper.java`

```java
public final class Looper { // 注意这里的final 关键字
    ...
     // sThreadLocal.get() will return null unless you've called prepare().
    @UnsupportedAppUsage
    static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
    @UnsupportedAppUsage
    private static Looper sMainLooper;  // guarded by Looper.class
    private static Observer sObserver;

    @UnsupportedAppUsage
    final MessageQueue mQueue;
    final Thread mThread;

}
```

`MessageQueue.java`

```java
public final class MessageQueue {
    ...
    Message message;
}
```



可以发现：

1. 一个线程，只有一个Looper
2. 一个Looper，只有一个MessageQueue
3. 一个MessageQueue有多个Message
4. 一个Message最多指定一个Handler来处理。

继续看源码，我们可以发现`Handler`主要这么两个职责：

- 处理`Message`
- 将某个`Message`发送出去（放到MessageQueue中）

来看看`Looper`取出消息分发的相关代码：

```java
public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();
		// 此处死循环来从队列里面取消息
        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                // 如果是空，说明正在退出
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            final Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }
            // Make sure the observer won't change while processing a transaction.
            final Observer observer = sObserver;

            final long traceTag = me.mTraceTag;
            long slowDispatchThresholdMs = me.mSlowDispatchThresholdMs;
            long slowDeliveryThresholdMs = me.mSlowDeliveryThresholdMs;
            if (thresholdOverride > 0) {
                slowDispatchThresholdMs = thresholdOverride;
                slowDeliveryThresholdMs = thresholdOverride;
            }
            final boolean logSlowDelivery = (slowDeliveryThresholdMs > 0) && (msg.when > 0);
            final boolean logSlowDispatch = (slowDispatchThresholdMs > 0);

            final boolean needStartTime = logSlowDelivery || logSlowDispatch;
            final boolean needEndTime = logSlowDispatch;

            if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
                Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
            }

            final long dispatchStart = needStartTime ? SystemClock.uptimeMillis() : 0;
            final long dispatchEnd;
            Object token = null;
            if (observer != null) {
                token = observer.messageDispatchStarting();
            }
            long origWorkSource = ThreadLocalWorkSource.setUid(msg.workSourceUid);
            try {
                // 最关键的地方，这里就是调用message的handler来处理这个消息。
                msg.target.dispatchMessage(msg);
                if (observer != null) {
                    observer.messageDispatched(token, msg);
                }
                dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;
            } catch (Exception exception) {
                if (observer != null) {
                    observer.dispatchingThrewException(token, msg, exception);
                }
                throw exception;
            } finally {
                ThreadLocalWorkSource.restore(origWorkSource);
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }
 

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycleUnchecked();
        }
    }
```

OK， 了解到了Looper会把Message分给Handler来做，那么来继续看Handler里面的`dispatchMessage(Message)`这个方法。

```java
public void dispatchMessage(@NonNull Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }
```

可以看到，这里的分发策略是：

:arrow_right:`Message.Callback`对象是否为空

​	在不为空的情况下，交由这个callback来处理，因此优先级较高

:arrow_right:`Handler.mCallback`

​	在不为空的情况下，交由这个callback来处理，优先级其次

:arrow_right:`handleMessage(Message)`

​	当上述条件都不满足的时候，调用它来处理。（通常，如果我们写一个子类来继承`Handler`类的话，你是需要重载这个方法的）

所以，`Handler`的扩展子类可以重载`dispatchMessage(Message)`这个方法，或者直接重载`handleMessage（message）`这个方法。具体情况，应该根据项目实际需求来走。



---



#### 进一步分析

看完上述代码之后，`Handler`这个哥们可能会让你疑惑，会有这么一个循环：

`Handler`:arrow_right:`MessageQueue`:arrow_right:`Message`:arrow_right:`Handler`

即：

1. `Handler`把消息放到`MessageQueue`中
2. `MessageQueue`的`Message`到`Handler`中

首先我们来看看`Handler`有哪些发消息的方法：

```java
// post系列
public final boolean post(@NonNull Runnable r);
public final boolean postAtTime(@NonNull Runnable r, long uptimeMillis);
public final boolean postAtTime(@NonNull Runnable r, @Nullable Object token, long uptimeMillis);
public final boolean postDelayed(@NonNull Runnable r, long delayMillis);
public final boolean postDelayed(@NonNull Runnable r, int what, long delayMillis);
public final boolean postDelayed(@NonNull Runnable r, Object token, long delayMillis);
public final boolean postAtFrontOfQueue(@NonNull Runnable r);

// send系列
public final boolean sendMessage(@NonNull Message msg)；
public final boolean sendEmptyMessage(int what)；
public final boolean sendEmptyMessageDelayed(int what, long delayMillis)；
public final boolean sendEmptyMessageAtTime(int what, long uptimeMillis)；
public final boolean sendMessageDelayed(@NonNull Message msg, long delayMillis)；
public boolean sendMessageAtTime(@NonNull Message msg, long uptimeMillis)；  
public final boolean sendMessageAtFrontOfQueue(@NonNull Message msg)；
    
```

这两个系列的方法都可以把`Message`放到`MessageQueue`中。不同的是，send系列参数都直接是`Message`, 而post系列参数都是`Runnable`,因此多了一步中间转换过程，然后再调用send系列的函数进行入队。

我们挑一个最简单的post系列函数，来分析一下源码：

```java
public final boolean post(Runnable r) {
    return  sendMessageDelayed(getPostMessage(r), 0);
}
```

可以发现，这里调用了一个函数`getPostMessage(Runnable)`来封装出一个`Message`,接着调用send系列的函数来入队：

```java
private static Message getPostMessage(Runnable r) {
        Message m = Message.obtain(); // 1
        m.callback = r; // 2
        return m;
    }
```

在代码1处，Android会维护一个全局的Message池，当用户需要使用Message的时候，可以直接通过obtain方法获得。这样的设计是避免不必要的资源浪费。

代码2处的直接把当前的runnable对象设置成回调函数。

当准备好Message之后，系统就会调用`sendMessageDelayed`方法来发送消息。这个函数可以设定多长时间的延迟再发送消息，内部的处理是通过当前时间+延迟时间算出该在哪个时间点发送（`sendMessageAtTime）`

```java
public final boolean sendMessageDelayed(@NonNull Message msg, long delayMillis) {
        if (delayMillis < 0) {
            delayMillis = 0;
        }
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
}
```

```java
public boolean sendMessageAtTime(@NonNull Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis); // 这里就是入队
    }
```

看到这里，我们会有一个疑问，为什么要大费周章，先把`Message` 放到`MessageQueue`里面，最后还是由`Handler`来处理呢？



---

#### 继续分析

上面设计哲学个人认为是有序性。

当你在主线程发送一个计算量特别大的任务的时候，如果你马上来处理这个任务，主线程就卡死了。那么交由子线程来计算，当它计算好了之后，发个消息告诉主线程，可以了。这样既不会影响主线程的交互（通常把UI线程称为主线程），计算任务又得以进行。

#### MessageQueue分析

MessageQueue的分析可能需要你具有一定的JNI知识。因此这里不会过多深入的分析。

简单看看源码，它有这么一些操作：

-  新建队列

  由一个本地方法（native方法）`nativeInit()`来创建完成，这个函数返回一个`long`类型的值。这个操作由指针操作完成，具体分析JNI的时候可以再回头看。

- 元素入队

  `boolean enqueueMessage(Message msg, long when)`

- 元素出队

  `Message next()`

- 删除元素

  `void removeMessages(Handler h, int what, Object obj)`

  `void removeMessages(Handler h, Runnable r, Object obj)`

- 销毁队列

  销毁队列也是一个本地方法完成的，`nativeDestroy()`。实质上，这个东西根所学的队列没有本质区别，因此不做过多赘述，有相关的需求可以回顾数据结构。



#### Looper分析

这个哥们比较有意思了。Looper有点像发动机的意思--它才会推动消息的处理。

应用程序使用Looper分两种情况，

:arrow_right_hook:主线程（MainThread，也是AcitivtyThread）

:arrow_right_hook:普通线程

我们看一下普通线程的用法：

```java
class MyLooperThread extends Thread {
    public Handler mhandler; // 谁干活？
    
    @Override
    public void run() {
        Looper.prepare(); // 1
        mHandler = new Handler() {
            public void handleMessage() {
                // 这里就是前文中处理消息的地方！！！
            }
        };
        Looper.loop(); // 2
    }
}
```

我们发现这里有三个步骤：

1. 调用Looper的`prepare()`方法；
2. 创建Handler；
3. Looper开始运作；

可以看到，我们没有发现有Looper对象的创建，最后也是一句Looper.loop，整个消息循环系统就跑起来了。

这是为什么呢？

很简单，我们来看看`prepare()`方法中干了些什么就知道了。

```java
public static void prepare() {
        prepare(true);
}

private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}

```



可以看到，它调用了另外一个prepare方法，在这里面，创建了一个新的Looper对象，封进ThreadLocal里面，看到了这里，你就明白，每个线程只有一个Looper对象是怎么实现的了把。然后，我们再回过头来看`Handler`的构造函数,就拿我们例子里面的空函数为例子：

```java
 public Handler(@Nullable Callback callback, boolean async) {
      ... //省略一些无关代码
        mLooper = Looper.myLooper(); // 这里本质上调用了ThreadLocal.get()方法返回当前线程的Looper对象。
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread " + Thread.currentThread()
                        + " that has not called Looper.prepare()"); // 这里的异常报错也提示我们，要先调用一次Looper.prepare()方法。
        }
        mQueue = mLooper.mQueue; // 设置MessageQueue
        mCallback = callback;
        mAsynchronous = async;
    }
```

到这里，整个的联系就盘活了。当前这个`Handler`执行发送消息的时候，就会把消息投送到这个`Looper`的`mQueue`里面。然后`Looper`里面的`loop()`方法被调用，就会不断的取出消息，接着调用`Handler`来进行处理。

---

从源码角度分析这几个东西的关系，逻辑就清晰很多，也感叹于谷歌工程师的天才设计！

