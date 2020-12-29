### AMS

#### AMS是什么？

它是Android提供的一个用于**管理Activity运行状态的系统进程**。我们的分析将按照这样的顺序进行：

1. AMS的功能
2. ActivityStack
3. ActivityTask

#### AMS的功能概述

和大多数系统服务一样，它是寄生在`SystemServer`里面的。当系统服务启动的时候，创建一个线程来处理客户的请求，如果到现在还不能理解的话，你可以想象成类似于socket那种监听模型，就是有一个serverSocket线程来监听进入的socket，从而开始处理请求。我们可以来看一下AMS的启动过程:

文件位置:`$CODEBASE/frameworks/base/services/java/com/android/server/SystemServer.java`

```java
public void run() {
    ...
    ActivityTaskManagerService atm = mSystemServiceManager.startService(
                ActivityTaskManagerService.Lifecycle.class).getService();
    mActivityManagerService = ActivityManagerService.Lifecycle.startService(
                mSystemServiceManager, atm); // 注意这里启动了AMS
    mActivityManagerService.setSystemServiceManager(mSystemServiceManager); // 向ServiceManager注册
    mActivityManagerService.setInstaller(installer);
    mWindowManagerGlobalLock = atm.getGlobalLock();
    ...
}
```

注意到这里调用了AMS内部类`LifeCycle`的`startService`方法，它本质上会调用`SystemServiceManager`的`startService`方法。这个`LifeCycle`类是`SystemService`的一个子类, 然后在`SystemServiceManager`这个类里面的的`startService`方法里面，调用了native方法来构造了一个新的`AMS$LifeCycle`的实例（走了它的构造函数），在`LifeCycle`的构造函数里面调用了`AMS`的构造函数。

文件位置：`$CODEBASE/frameworks/base/services/core/java/com/android/server/SystemServiceManager.java`

```java
public <T extends SystemService> T startService(Class<T> serviceClass) {
    ...
    final T service;
    try {
        Constructor<T> constructor = serviceClass.getConstructor(Context.class);
        service = constructor.newInstance(mContext); // 这个newInstance方法是native方法
    }catch(Exception ex) {
        ...
    }
    startService(service); // 走接下来的重载方法
}
```

```java
public void startService(@NonNull final SystemService service) {
        // Register it.
        mServices.add(service); // 注意这里，把AMS$LifeCycle加入了一个ArrayList里面进行管理。
        // Start it.
        long time = SystemClock.elapsedRealtime();
        try {
            service.onStart(); // 回调AMS$LifeCycle的onStart方法
        } catch (RuntimeException ex) {
            throw new RuntimeException("Failed to start service " + service.getClass().getName()
                    + ": onStart threw an exception", ex);
        }
        warnIfTooLong(SystemClock.elapsedRealtime() - time, service, "onStart");
    }
```

可以看到，它又跑回去调用`AMS$LifeCycle`的`onStart`方法了。在AMS$LifeCycle的`onStart`方法里面，走了AMS的`start`方法，这个方法里面主要就是创建和启动AMS线程，而且，这个线程是必须启动成功的，可以试着想想看，如果AMS拉不起来，你的设备就算其他服务启动了，也没啥意义。这个AMS也是一个Binder Server，因此，要查看它的功能，其实可以去翻看一下自动编译生成的`IActiivityManager.java`这个文件。这个文件有很多行，提供的功能也很多，我们这里大概看看几个重要的功能。

- 组件状态管理

  这里的组件指的是四大组件，状态管理包括关闭，开启一系列的相关操作。如启动Activity等。

- 组件状态查询

  这里就是指获取组件的信息，如getCallingActivity等。

- Task相关

- 其他杂项功能

#### ActivityStack

文件位置：`$CODEBASE/frameworks/base/services/core/java/com/android/server/wm/ActivityStack.java`

这个东西叫做活动栈，这个哥们是管理**系统所有活动的Activity状态**的数据结构。首先，大家都可能了解栈这个数据结构，首先问一个问题：如何用多个ArrayList来模拟栈的功能？留个心眼，我们继续往前走。

1. ActivityState

   它描述了一个Activity可能经历的所有状态：

   ```java
    enum ActivityState {
           INITIALIZING,
           RESUMED,
           PAUSING,
           PAUSED,
           STOPPING,
           STOPPED,
           FINISHING,
           DESTROYING,
           DESTROYED,
           RESTARTING_PROCESS
       }
   ```

   一个Activity会经历上述的一些状态,这些状态对应的是Activity的生命周期方法，大家可以回去翻看API文档的那7个生命周期回调方法。
   
2. ArrayList

   ActivityStack管理了一系列的ArrayList， 这些ArrayList保存的东西都是ActivityRecord。ActivityRecord所对应的是，一个Activity就对应了一个AcitivtyRecord。

   ```java
       ... 
       /**
        * The back history of all previous (and possibly still
        * running) activities.  It contains #TaskRecord objects.
        */
       private final ArrayList<TaskRecord> mTaskHistory = new ArrayList<>();
   
       /**
        * List of running activities, sorted by recent usage.
        * The first entry in the list is the least recently used.
        * It contains HistoryRecord objects.
        */
       private final ArrayList<ActivityRecord> mLRUActivities = new ArrayList<>();
   
       /**
        * When we are in the process of pausing an activity, before starting the
        * next one, this variable holds the activity that is currently being paused.
        */
       ActivityRecord mPausingActivity = null;
   
       /**
        * This is the last activity that we put into the paused state.  This is
        * used to determine if we need to do an activity transition while sleeping,
        * when we normally hold the top activity paused.
        */
       ActivityRecord mLastPausedActivity = null;
   
       /**
        * Activities that specify No History must be removed once the user navigates away from them.
        * If the device goes to sleep with such an activity in the paused state then we save it here
        * and finish it later if another activity replaces it on wakeup.
        */
       ActivityRecord mLastNoHistoryActivity = null;
   
       /**
        * Current activity that is resumed, or null if there is none.
        */
       ActivityRecord mResumedActivity = null;
   ...
   ```

   这里我们可以做个实验：

   在`ActivityStack`的`startActivityLocked()`方法中添加Log打印:

   ```JAVA
   void startActivityLocked(ActivityRecord r, ActivityRecord focusedTopActivity,
               boolean newTask, boolean keepCurTransition, ActivityOptions options) {
           if (r != null && focusedTopActivity != null) {
   
               String msg = "we enter startActivityLocked() method!\nActivityRecord = " + r.toString() + ": ActivityRecord(focusedTopActivity) = " + focusedTopActivity
                       .toString();
               Log.d("TESTTAG", msg);
           }
           ...
   }
   ```

   编译代码然后烧录镜像（或者冷启动一下你的模拟机）

   ```shell
   $> source build/envsetup.sh
   $> lunch xx # xx 为你的combo
   $> make -j48 && emulator -no-snapshot-load
   ```

   

   然后我们编写一个Activity，里面有个按钮，自己启动自己：

   ```java
   import android.app.Activity;
   import android.content.Intent;
   import android.os.Bundle;
   import android.widget.Button;
   
   public class MainActivity extends Activity {
   
       private Button btn;
   
   
       @Override
       protected void onCreate(Bundle savedInstanceState) {
           super.onCreate(savedInstanceState);
   
           setContentView(R.layout.activity_main);
           btn = findViewById(R.id.button);
           btn.setOnClickListener(v -> startActivity(new Intent(getApplicationContext(), 			MainActivity.class)));
       }
   }
   ```

   我们所做的操作是，点击这个按钮两次，那么按道理说，此时应该有三个MainActivity的实例（因为启动模式是standard）。我们来看Logcat给出的输出：

   ```java
   2020-12-29 19:00:26.281 1641-2595/system_process D/TESTTAG: we enter startActivityLocked() method!
       ActivityRecord = ActivityRecord{eb0b8f1 u0 com.example.myapplication1/.MainActivity t28}: ActivityRecord(focusedTopActivity) = ActivityRecord{9e899cf u0 com.android.launcher3/.Launcher t27} // 这里的t27的意思是task ID 是27
    // 从这条记录我们可以发现，我们从Launcher启动我们的Activity的时候，创建了一个新的ActivityRecord记录。
   // 注意观察这里的ActivityRecord的hash值的变化。
   // 注意观察TaskID的变化。
   
   2020-12-29 19:00:36.342 1641-2137/system_process D/TESTTAG: we enter startActivityLocked() method!
       ActivityRecord = ActivityRecord{4118cff u0 com.example.myapplication1/.MainActivity t28}: ActivityRecord(focusedTopActivity) = ActivityRecord{eb0b8f1 u0 com.example.myapplication1/.MainActivity t28}
   // 因为启动模式是standard，因此继续启动新的MainActivity实例。
   
   2020-12-29 19:01:03.114 1641-1979/system_process D/TESTTAG: we enter startActivityLocked() method!
       ActivityRecord = ActivityRecord{6236e55 u0 com.example.myapplication1/.MainActivity t28}: ActivityRecord(focusedTopActivity) = ActivityRecord{4118cff u0 com.example.myapplication1/.MainActivity t28}
   
   
   ```

   我们可以发现，系统如实的记录了我们的启动顺序：

   `LauncherActivity(9e899cf)` -> `MainActivity1(eb0b8f1)`

   `MainActivity1(eb0b8f1)` -> `MainActivity2(4118cff)`

   `MainActivity2(4118cff)`-> `MainActivity3(6236e55)`

所以，我们可以得出这么一个结论：

**AMS是通过ActivityStack来管理，记录系统中的Activity和其他组件的状态，同时提供查询功能的一个系统服务**



我是一个调皮的分割线

---

那么，我们走到这里，下一个要回答的问题是：

**一个Activity是怎么被启动的？**

我们从最直观的调用入口开始：

`startActivity(Intent intent)`

这个东西大家都不会陌生，它用于启动一个目标Activity---具体启动哪个，由AMS通过对系统所有的程序进行`intent`的匹配得到，不局限于当前package的范围。因此，`startActivity`很可能启动的不是本应用的组件。

我们可以通过大致跟踪一下startActivity的调用流程：

1. 在Activity里面调用`startActivity`方法， 最终会调用到 `execStartActivity@Instrumentation`

2. 深入到`Instrumentation.java`来看，它通过RPC调用了`ActivityTaskManagerService`的`startActivity`方法。

3. 注意，现在我们移步进入了`ActivityTaskManagerService`，它调用了这样的一个方法：

   `startActivityAsUser()`，来看看这两个方法的签名区别：

   ```java
   public final int startActivity(IApplicationThread caller, String callingPackage,Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,int startFlags, ProfilerInfo profilerInfo, Bundle bOptions)
       
   public int startActivityAsUser(IApplicationThread caller, String callingPackage,Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,int startFlags, ProfilerInfo profilerInfo, Bundle bOptions, int userId) 
   ```

   可以看到，他们两个方法唯一区别多了一个参数 `userId`。这个通过`UserHandle.getCallingUserId()`方法拿到。顺口提一句，这里也是通过Binder拿的用户UID。这个UID在后面用来判定权限相关的工作。

4. 来到这个方法：

   ```java
   int startActivityAsUser(IApplicationThread caller, String callingPackage,
               Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
               int startFlags, ProfilerInfo profilerInfo, Bundle bOptions, int userId,
               boolean validateIncomingUser) {
           enforceNotIsolatedCaller("startActivityAsUser");
   
           userId = getActivityStartController().checkTargetUser(userId, validateIncomingUser,
                   Binder.getCallingPid(), Binder.getCallingUid(), "startActivityAsUser");
   
           // TODO: Switch to user app stacks here.
           return getActivityStartController().obtainStarter(intent, "startActivityAsUser") // 链式调用，这个地方获取一个叫ActivityStarter的一个对象，来启动Acitivty。注意看这里的参数。
                   .setCaller(caller)
                   .setCallingPackage(callingPackage)
                   .setResolvedType(resolvedType)
                   .setResultTo(resultTo)
                   .setResultWho(resultWho)
                   .setRequestCode(requestCode)
                   .setStartFlags(startFlags)
                   .setProfilerInfo(profilerInfo)
                   .setActivityOptions(bOptions)
                   .setMayWait(userId)
                   .execute(); 
   
       }
   ```

   

