###　安卓当中的同步机制

我们先回忆一下，操作系统中有哪些同步的手段？

- 信号量（Semaphore）

  它主要包含两个操作，P和V。 Semaphore 来指示这个共享资源的可用数量，换种理解是，有多少人可以使用这个资源？

  P操作可以减少信号量的计数，V操作可以增加计数。P V 这两个操作都是属于原子操作，意味着执行过程不可以被中断。

- 互斥量（Mutex）

  简单而言，你可以认为他是简化版的信号量。也就是取值只能为0和1的信号量（Binary Semaphore）。换句话说，操作系统很多时候的资源都具有排他性--这个资源当前要么被占用，要么可以被访问。Mutex相较于Semaphore实现起来也更为简单。

- 管程（Monitor）

- Futex（Fast Userspace mutexs）Linux独有的，好像在早期的某个内核版本被加入。



那么我们来看看Android中有哪些手段吧。

#### Mutex

文件位置：`$ANDROID_CODEBASE/system/core/libutils/include/utils/Mutex.h`,如果下面没有特殊说明，默认我们的路径都是从`$ANDROID_CODEBASE`开始。它表示你的源码根目录。

为了简单，我就摘取一些相关的代码，其他代码有兴趣的可以自行深入分析。

```c++
class CAPABILITY("mutex") Mutex {
  public:
    enum {
        PRIVATE = 0, // 只支持同一个进程间的同步
        SHARED = 1 // 支持跨进程间的同步
    };

    Mutex();
    explicit Mutex(const char* name);
    explicit Mutex(int type, const char* name = nullptr);
    ~Mutex();

    // lock or unlock the mutex
    status_t lock() ACQUIRE();
    void unlock() RELEASE();

    // lock if possible; returns 0 on success, error otherwise
    status_t tryLock() TRY_ACQUIRE(0);
    ...
```



两个枚举量的意思我已经注释中写明白了，这里只有三个有意思的方法，

```c++
status_t lock();
void unlock();
status_t tryLock();
```

从名字上我们可以看到他们的作用，我们来进一步看看他的实现：

```c++
inline status_t Mutex::lock() {
    return -pthread_mutex_lock(&mMutex);
}
inline void Mutex::unlock() {
    pthread_mutex_unlock(&mMutex);
}
inline status_t Mutex::tryLock() {
    return -pthread_mutex_trylock(&mMutex);
}
```

这里你就会发现，他本质上其实是对pthread的接口做了一些相关的封装 。

#### Condition

这个东西字面意思是条件。他的设计哲学是，判断一个条件是不是满足了？—— 如果满足了，那就返回，继续执行下去，否则就休眠等待。**直到条件被满足**。

退一步想想，这种情况能用Mutex做么？ 理论上应该可以的。举个例子，假设我们两个线程`A` ，` B`， 他们会同时修改一个全局变量`var`, 并且我们定义行为：

`Thread A` 不断修改 `var`的值，每次改变之后的值是未知的

`Thread B`观察`var`的值，如果为0了的时候执行某些动作。

我们可以看到， A ， B 两个线程都想访问`var`这个资源。Mutex是个思路。但是我们想想，线程B 等待的是当 `var`这个变量为0的情况，醉翁之意不在酒。

如果用Mutex写，类似于这样的：

```c
while (1) { // 死循环
    acquire_mutex_lock(var); // 去获取mutex锁
    if (var == 0) { // 条件满足
        release_mutex_lock(var);
        break;
    }else {
        release_mutex_lock(var); // 下一轮我们再看看
        sleep();
    }
}
```

所以我们可以看到，这种轮询方式特别耗费CPU时间。

举个例子，假如有两个角色，厕所维护员还有使用厕所的用户，代表上述A B 两个角色。然后我们把厕纸当做变量var。那么出现这种场景，用户随便用厕纸，厕纸的余量是多少未知。但是工作人员等厕纸余量为0的时候，需要去更换厕纸。Mutex的机制下，可以看到，工作人员和普通拉屎的用户一样，都要排队轮询进厕所来看看厕纸。那么可以看到，这个工作人员效率很低，他要跟大家一起排队来获取进入厕所的机会。

那么有一种思路是，工作人员不排队，当厕纸用完了，有个人通知他，叫他进来换厕纸，减少排队数量，提高效率。

Condition就是来解决这类问题的。

- 文件位置：`system/core/libutils/include/utils/Condition.h`

```c++
class Condition {
public:
    enum {
        PRIVATE = 0, // 和前面类似，也有跨进程共享的支持
        SHARED = 1
    };

    enum WakeUpType {
        WAKE_UP_ONE = 0,
        WAKE_UP_ALL = 1
    };

    Condition();
    explicit Condition(int type);
    ~Condition();
    // Wait on the condition variable.  Lock the mutex before calling.
    // Note that spurious wake-ups may happen.
    status_t wait(Mutex& mutex); // 在某个条件上等待
    // same with relative timeout
    status_t waitRelative(Mutex& mutex, nsecs_t reltime); // 同上，但是这里是超时退出
    // Signal the condition variable, allowing one thread to continue.
    void signal(); // 满足条件了，通知等待者
    // Signal the condition variable, allowing one or all threads to continue.
    void signal(WakeUpType type) {
        if (type == WAKE_UP_ONE) {
            signal();
        } else {
            broadcast();
        }
    }
    // Signal the condition variable, allowing all threads to continue.
    void broadcast(); // 条件满足时 通知所有等待者
```

其实如果你英文好的话，看上面的注释你就可以明白了。

这里是个C++的类，那么我们来看看这里的一些关键实现方法。

```c++
inline status_t Condition::wait(Mutex& mutex) {
    return -pthread_cond_wait(&mCond, &mutex.mMutex);
}

inline status_t Condition::waitRelative(Mutex& mutex, nsecs_t reltime) {
    struct timespec ts;
#if defined(__linux__)
    clock_gettime(CLOCK_MONOTONIC, &ts); // linux和apple下获取时间的函数，编译相关
#else // __APPLE__
    // Apple doesn't support POSIX clocks.
    struct timeval t;
    gettimeofday(&t, nullptr);
    ts.tv_sec = t.tv_sec;
    ts.tv_nsec = t.tv_usec*1000;
#endif

    // On 32-bit devices, tv_sec is 32-bit, but `reltime` is 64-bit.
    int64_t reltime_sec = reltime/1000000000;

    ts.tv_nsec += static_cast<long>(reltime%1000000000);
    if (reltime_sec < INT64_MAX && ts.tv_nsec >= 1000000000) {
        ts.tv_nsec -= 1000000000;
        ++reltime_sec;
    }

    int64_t time_sec = ts.tv_sec;
    if (time_sec > INT64_MAX - reltime_sec) {
        time_sec = INT64_MAX;
    } else {
        time_sec += reltime_sec;
    }

    ts.tv_sec = (time_sec > LONG_MAX) ? LONG_MAX : static_cast<long>(time_sec);

    return -pthread_cond_timedwait(&mCond, &mutex.mMutex, &ts);
}


inline void Condition::signal() {
    pthread_cond_signal(&mCond);
}
inline void Condition::broadcast() {
    pthread_cond_broadcast(&mCond);
}

```

可以看到，还是pthread的相关接口的封装。

这里留个问题，为什么wait函数的参数还是要用到Mutex呢？

#### Barrier

意思是屏障。接上文我们继续思考，我们来看一个Condition的例子。

Barrier这个东西是为了SurfaceFlinger这个东西设计的，不像Mutex，Condition一样作为Util工具来提供给大家用。不过我们来看看Barrier这个例子。

文件位置：`frameworks/native/services/surfaceflinger/Barrier.h`

```c++
class Barrier
{
public:
    // Release any threads waiting at the Barrier.
    // Provides release semantics: preceding loads and stores will be visible
    // to other threads before they wake up.
    void open() {
        std::lock_guard<std::mutex> lock(mMutex);
        mIsOpen = true;
        mCondition.notify_all();
    }

    // Reset the Barrier, so wait() will block until open() has been called.
    void close() {
        std::lock_guard<std::mutex> lock(mMutex);
        mIsOpen = false;
    }

    // Wait until the Barrier is OPEN.
    // Provides acquire semantics: no subsequent loads or stores will occur
    // until wait() returns.
    void wait() const {
        std::unique_lock<std::mutex> lock(mMutex);
        mCondition.wait(lock, [this]() NO_THREAD_SAFETY_ANALYSIS { return mIsOpen; });
    }
private:
    mutable std::mutex mMutex;
    mutable std::condition_variable mCondition;
    int mIsOpen GUARDED_BY(mMutex){false};
};

```

可以看到，有三个函数，分别是`wait`, `close`,`open`。

既然说它是condition的一个例子，那么barrier等待的条件是什么呢？

答案是 `mIsOpen`。我们来看`wait`函数，他首先获取了一个mutex锁，然后才去调用Condition的`wait`。为什么呢？因为`mIsOpen`这个变量如果没有被互斥锁保护起来的话，`open/close`同时去操作他的话，会发生什么情况呢？

所以这就是整个设计的一个巧妙的地方。