### Zygote的启动

如果你没有看过[Android启动过程分析](Android/Launcher.md)的话，建议你先看看那里面讲的东西。

好了，接着上文所讲，我们来看看zygote进程的源码：

```cpp
int main(int argc, char* const argv[])
{
  ... // 省略一些不必要的调试日志代码
    AppRuntime runtime(argv[0], computeArgBlockSize(argc, argv));
    // Process command line arguments
    // ignore argv[0]
    argc--;
    argv++;
	// 注意看这里的注释，这里的注释基本上有很多解释。
    // Everything up to '--' or first non '-' arg goes to the vm.
    //
    // The first argument after the VM args is the "parent dir", which
    // is currently unused.
    //
    // After the parent dir, we expect one or more the following internal
    // arguments :
    // 参数名字
    // --zygote : Start in zygote mode 以zygote模式启动
    // --start-system-server : Start the system server. 启动系统服务
    // --application : Start in application (stand alone, non zygote) mode. 应用模式启动
    // --nice-name : The nice name for this process. 别名
    //
    // For non zygote starts, these arguments will be followed by
    // the main class name. All remaining arguments are passed to
    // the main method of this class.
    //
    // For zygote starts, all remaining arguments are passed to the zygote.
    // main function.
    //
    // Note that we must copy argument string values since we will rewrite the
    // entire argument block when we apply the nice name to argv0.
    //
    // As an exception to the above rule, anything in "spaced commands"
    // goes to the vm even though it has a space in it.
    const char* spaced_commands[] = { "-cp", "-classpath" }; // 这里指定了classpath，java启动需要这个。 记住这里
    // Allow "spaced commands" to be succeeded by exactly 1 argument (regardless of -s).
    bool known_command = false;

    int i;
    for (i = 0; i < argc; i++) {
        if (known_command == true) {
          runtime.addOption(strdup(argv[i]));
          // The static analyzer gets upset that we don't ever free the above
          // string. Since the allocation is from main, leaking it doesn't seem
          // problematic. NOLINTNEXTLINE
          ALOGV("app_process main add known option '%s'", argv[i]);
          known_command = false;
          continue;
        }

        for (int j = 0;
             j < static_cast<int>(sizeof(spaced_commands) / sizeof(spaced_commands[0]));
             ++j) {
          if (strcmp(argv[i], spaced_commands[j]) == 0) {
            known_command = true;
            ALOGV("app_process main found known command '%s'", argv[i]);
          }
        }

        if (argv[i][0] != '-') {
            break;
        }
        if (argv[i][1] == '-' && argv[i][2] == 0) {
            ++i; // Skip --.
            break;
        }

        runtime.addOption(strdup(argv[i]));
        // The static analyzer gets upset that we don't ever free the above
        // string. Since the allocation is from main, leaking it doesn't seem
        // problematic. NOLINTNEXTLINE
        ALOGV("app_process main add option '%s'", argv[i]);
    }

    // Parse runtime arguments.  Stop at first unrecognized option.
    bool zygote = false;
    bool startSystemServer = false;
    bool application = false;
    String8 niceName;
    String8 className;

    ++i;  // Skip unused "parent dir" argument.
    while (i < argc) {
        const char* arg = argv[i++];
        if (strcmp(arg, "--zygote") == 0) { // 判断当前是否是zygote模式启动
            zygote = true;
            niceName = ZYGOTE_NICE_NAME;
        } else if (strcmp(arg, "--start-system-server") == 0) {
            startSystemServer = true;
        } else if (strcmp(arg, "--application") == 0) {
            application = true;
        } else if (strncmp(arg, "--nice-name=", 12) == 0) {
            niceName.setTo(arg + 12);
        } else if (strncmp(arg, "--", 2) != 0) {
            className.setTo(arg);
            break;
        } else {
            --i;
            break;
        }
    }

	.... // 省略不必要的代码
    if (zygote) {
        runtime.start("com.android.internal.os.ZygoteInit", args, zygote); // 最后调用runtime.start。
    } else if (className) {
        runtime.start("com.android.internal.os.RuntimeInit", args, zygote);
    } else {
        fprintf(stderr, "Error: no class name or --zygote supplied.\n");
        app_usage();
        LOG_ALWAYS_FATAL("app_process: no class name or --zygote supplied.");
    }
}

```

那么这个runtime是什么？

它实质上是个AndroidRuntime对象。

它的start源码如下：

代码位置：`$ANDROID_CODEBASE/frameworks/base/core/jni/AndroidRuntime.cpp`

```cpp
/*
 * Start the Android runtime.  This involves starting the virtual machine
 * and calling the "static void main(String[] args)" method in the class
 * named by "className".
 *
 * Passes the main function two arguments, the class name and the specified
 * options string.
 */
void AndroidRuntime::start(const char* className, const Vector<String8>& options, bool zygote)
{
  ... // 省略不必要的代码
    /* start the virtual machine */
    JniInvocation jni_invocation;
    jni_invocation.Init(NULL);
    JNIEnv* env;
    if (startVm(&mJavaVM, &env, zygote) != 0) {
        return; // 注意这里启动了虚拟机，这里就是Java世界啦！
    }
    onVmCreated(env); // 虚拟机启动的回调

	... // 省略不必要的代码
}
```

日后我们有时间分析虚拟机的启动，这里假设成功启动，我们进入ZygoteInit的启动:

代码位置：`$ANDROID_CODE_BASE/frameworks/base/core/java/com/android/internal/os/ZygoteInit.java`

```java
  @UnsupportedAppUsage
    public static void main(String argv[]) {
        ZygoteServer zygoteServer = null;

        // Mark zygote start. This ensures that thread creation will throw
        // an error.
        ZygoteHooks.startZygoteNoThreadCreation();

        // Zygote goes into its own process group.
        try {
            Os.setpgid(0, 0);
        } catch (ErrnoException ex) {
            throw new RuntimeException("Failed to setpgid(0,0)", ex);
        }

        Runnable caller;
        try {
            // Report Zygote start time to tron unless it is a runtime restart
            if (!"1".equals(SystemProperties.get("sys.boot_completed"))) {
                MetricsLogger.histogram(null, "boot_zygote_init",
                        (int) SystemClock.elapsedRealtime());
            }

            String bootTimeTag = Process.is64Bit() ? "Zygote64Timing" : "Zygote32Timing";
            TimingsTraceLog bootTimingsTraceLog = new TimingsTraceLog(bootTimeTag,
                    Trace.TRACE_TAG_DALVIK);
            bootTimingsTraceLog.traceBegin("ZygoteInit");
            RuntimeInit.enableDdms();

            boolean startSystemServer = false;
            String zygoteSocketName = "zygote";
            String abiList = null;
            boolean enableLazyPreload = false;
            for (int i = 1; i < argv.length; i++) {
                if ("start-system-server".equals(argv[i])) { // 前面的这个参数一直被传递到这里。
                    startSystemServer = true;
                } else if ("--enable-lazy-preload".equals(argv[i])) {
                    enableLazyPreload = true;
                } else if (argv[i].startsWith(ABI_LIST_ARG)) {
                    abiList = argv[i].substring(ABI_LIST_ARG.length());
                } else if (argv[i].startsWith(SOCKET_NAME_ARG)) {
                    zygoteSocketName = argv[i].substring(SOCKET_NAME_ARG.length());
                } else {
                    throw new RuntimeException("Unknown command line argument: " + argv[i]);
                }
            }

            final boolean isPrimaryZygote = zygoteSocketName.equals(Zygote.PRIMARY_SOCKET_NAME);

            if (abiList == null) {
                throw new RuntimeException("No ABI list supplied.");
            }

            // In some configurations, we avoid preloading resources and classes eagerly.
            // In such cases, we will preload things prior to our first fork.
            if (!enableLazyPreload) {
                bootTimingsTraceLog.traceBegin("ZygotePreload");
                EventLog.writeEvent(LOG_BOOT_PROGRESS_PRELOAD_START,
                        SystemClock.uptimeMillis());
                preload(bootTimingsTraceLog); // 预加载各类资源
                EventLog.writeEvent(LOG_BOOT_PROGRESS_PRELOAD_END,
                        SystemClock.uptimeMillis());
                bootTimingsTraceLog.traceEnd(); // ZygotePreload
            } else {
                Zygote.resetNicePriority();
            }

            // Do an initial gc to clean up after startup
            bootTimingsTraceLog.traceBegin("PostZygoteInitGC");
            gcAndFinalize();
            bootTimingsTraceLog.traceEnd(); // PostZygoteInitGC

            bootTimingsTraceLog.traceEnd(); // ZygoteInit
            // Disable tracing so that forked processes do not inherit stale tracing tags from
            // Zygote.
            Trace.setTracingEnabled(false, 0);


            Zygote.initNativeState(isPrimaryZygote);

            ZygoteHooks.stopZygoteNoThreadCreation();

            zygoteServer = new ZygoteServer(isPrimaryZygote); // 这里会去创建一个serverSocket.

            if (startSystemServer) {
                Runnable r = forkSystemServer(abiList, zygoteSocketName, zygoteServer); // 干活的地方出现了！这里就是启动的地方。

                // {@code r == null} in the parent (zygote) process, and {@code r != null} in the
                // child (system_server) process.
                if (r != null) {
                    r.run();
                    return;
                }
            }

            Log.i(TAG, "Accepting command socket connections");

            // The select loop returns early in the child process after a fork and
            // loops forever in the zygote.
            caller = zygoteServer.runSelectLoop(abiList);
        } catch (Throwable ex) {
            Log.e(TAG, "System zygote died with exception", ex);
            throw ex;
        } finally {
            if (zygoteServer != null) {
                zygoteServer.closeServerSocket();
            }
        }

        // We're in the child process and have exited the select loop. Proceed to execute the
        // command.
        if (caller != null) {
            caller.run();
        }
    }
```

可以发现，ZygoteInit这里面其实并不复杂，完成了两个工作：

- 注册一个Socket

  Zygote是一个总管家的角色，因此有新程序需要运行时，系统通过这个socket来通知总管家，并且由他来进行进程的孵化。

- 预加载各类资源。

  ```java
      static void preload(TimingsTraceLog bootTimingsTraceLog) {
         
          beginPreload(); 
         
          preloadClasses();
    
          preloadResources();
        
          nativePreloadAppProcessHALs();
         
          maybePreloadGraphicsDriver();
         
          preloadSharedLibraries();
          
          preloadTextResources();
          // Ask the WebViewFactory to do any initialization that must run in the zygote process,
          // for memory sharing purposes.
          WebViewFactory.prepareWebViewInZygote();
          endPreload();
          warmUpJcaProviders();
          Log.d(TAG, "end preload");
  
          sPreloadComplete = true;
      }
  ```

  

不难从方法名里面看出加载了哪些东西。举个例子，以`preloadClasses()`为例子，它加载了一些常用的类， 这些类

被记录在`/system/etc/preload-classes`这个文件里面。

```java
android.R$styleable
android.accessibilityservice.AccessibilityServiceInfo$1
android.accessibilityservice.AccessibilityServiceInfo
android.accounts.Account$1
android.accounts.Account
android.accounts.AccountManager$10
android.accounts.AccountManager$11
android.accounts.AccountManager$18
android.accounts.AccountManager$1
android.accounts.AccountManager$20
android.accounts.AccountManager$2
android.accounts.AccountManager$AmsTask$1
android.accounts.AccountManager$AmsTask$Response
android.accounts.AccountManager$AmsTask
android.accounts.AccountManager$BaseFutureTask$1
android.accounts.AccountManager$BaseFutureTask$Response
android.accounts.AccountManager$BaseFutureTask
android.accounts.AccountManager$Future2Task$1
android.accounts.AccountManager$Future2Task
android.accounts.AccountManager
android.accounts.AccountManagerCallback
android.accounts.AccountManagerFuture
android.accounts.AccountsException
...
```

里面记录了上千个类，包含了一些常用的系统资源类，

这个文件由`$ANDROID_CODEBASE/frameworks/base/tools/preload/WritePreloadedClassFile.java`这个文件自动生成，这个就是涉及到文件的读写，感兴趣的自行阅读，不作深入分析。

#### SystemServer的启动

还记得有个参数`--start-system-server`么？如果调用的时候还有这个参数的话，那么还要启动SystemServer。它的启动是交由`forkSystemServer()`这个函数来启动，我们稍后分析这个函数。

我们来考虑这么一个问题，Zygote前期负责启动系统服务，后面还要启动程序（孵化程序），但是，这个玩意只会在`init.rc`里面启动一次，它是怎么完成这两个工作呢？我们推测，`forkSystemServer()`会启动一个专门的进程来承载服务的运行，然后`app_process`的进程就负责孵化（当所有应用程序的爹）。是这样么？

还是得去看看代码：

