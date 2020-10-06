### ActivityThread分析

#### 他是个啥

首先问大家一个问题，一个Activity的生命周期方法有哪些？

不多说，上图：

![Activity生命周期](../img/activity_lifecycle.png)

我们熟悉的一个Java应用，不应该是从

```java
public static void main(String args[]) {
    ....
}
```

开始吗？那么我们这里的main方法那里去了？答案就在这个ActivityThread里面。

#### 代码分析

这个哥们在哪里呢？

`$ANDROID_BASE/frameworks/base/core/java/android/app/ActivityThread.java`



我们在这个类里面，可以发现他的main方法：

```java
public static void main(String[] args) {
        Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");

        // Install selective syscall interception
        AndroidOs.install();

        // CloseGuard defaults to true and can be quite spammy.  We
        // disable it here, but selectively enable it later (via
        // StrictMode) on debug builds, but using DropBox, not logs.
        CloseGuard.setEnabled(false);

        Environment.initForCurrentUser();

        // Make sure TrustedCertificateStore looks in the right place for CA certificates
        final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
        TrustedCertificateStore.setDefaultUserDirectory(configDir);

        Process.setArgV0("<pre-initialized>");
		// 这里调用了Looper的prepare方法 
        Looper.prepareMainLooper();

        // Find the value for {@link #PROC_START_SEQ_IDENT} if provided on the command line.
        // It will be in the format "seq=114"
        long startSeq = 0;
        if (args != null) {
            for (int i = args.length - 1; i >= 0; --i) {
                if (args[i] != null && args[i].startsWith(PROC_START_SEQ_IDENT)) {
                    startSeq = Long.parseLong(
                            args[i].substring(PROC_START_SEQ_IDENT.length()));
                }
            }
        }
    	// 创建一个ActivityThread实例
        ActivityThread thread = new ActivityThread();
        thread.attach(false, startSeq);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        if (false) {
            
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        // End of event ActivityThreadMain.
        Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
    	// 可以开始处理消息了
        Looper.loop();

        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
```

比较一下这个代码和我们前文[Android Looper的分析](Android/LooperHandler.md)里面的LooperThread例子就可以知道，架构是一样的