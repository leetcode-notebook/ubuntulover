### Acitivity的启动模式

Activity是我们和安卓开发打交道的入门的东西了。我们都知道，启动activity的方法是从startActivity开始，因此有必要分析一下Activity的启动模式。

它总共有这么四种启动模式：

- `standard`
- `singleTop`
- `singleTask`
- `singleInstance`

接下来，我们一边看理论，一边结合实际例子来分析这四种启动模式的特点。

首先，为了方便查看打印信息，我们首先来定义一个`BaseActivity`,里面的内容是在`onCreate`方法和`onNewIntent`方法中加入打印信息，打印当前所属的`Task`, 当前类的`hashCode`,以及`taskAffinity`的值，后续我们的测试`Activity`都继承该类，不再赘述。

放代码：

```java
package com.example.myapplication;

import android.content.Intent;
import android.content.pm.ActivityInfo;
import android.content.pm.PackageManager;
import android.os.Bundle;
import android.util.Log;

import androidx.appcompat.app.AppCompatActivity;

public class BaseActivity extends AppCompatActivity {


    private static final String LOG_TAG = "LOG_TAG";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Log.i(LOG_TAG, "onCreate method !");
        Log.i(LOG_TAG, "onCreate" + getClass().getSimpleName() + " TaskId = " + getTaskId() + " hashCode = " + this.hashCode());
        dumpTaskAffinity();
    }

    @Override
    protected void onNewIntent(Intent intent) {
        super.onNewIntent(intent);
        Log.i(LOG_TAG, "onNewIntent method !");
        Log.i(LOG_TAG, "onNewIntent" + getClass().getSimpleName() + " TaskId = " + getTaskId() + " hashCode = " + this.hashCode());
    }

    protected void dumpTaskAffinity() {

        try {
            ActivityInfo info = this.getPackageManager().getActivityInfo(getComponentName(), PackageManager
                    .GET_META_DATA);
            Log.i(LOG_TAG, "taskAffinity : " + info.taskAffinity);
        } catch (PackageManager.NameNotFoundException e) {
            e.printStackTrace();
        }
    }


}
```

#### standard 标准模式

这个模式是默认的启动模式。不指定任何启动模式的情况下，系统就默认使用这种模式启动Activity。**每次启动一个`Activity`都重新创建一个实例**， 不管这个实例存不存在，这种模式下，谁启动了该模式的`Activity`,  这个`Activity`就属于谁的任务栈。我们在`AndroidManifest.xml`这个文件里面不做任何修改或者加这么一句即可：

```xml
<activity
            android:name=".StandardActivity"
            android:launchMode="standard">
</activity>
```





我们来看例子：

```java
package com.example.myapplication;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;

public class StandardActivity extends BaseActivity {


    Button btn;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_standard);
        btn = findViewById(R.id.btn_jmp);
        btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent(StandardActivity.this, StandardActivity.class);
                startActivity(intent);
            }
        });
    }
}
```

这里代码很简单，只有一个按钮，因此不去给出布局文件。这里我们点击**四次**按钮， 再按**四次**返回键。

观察打印输出：

```
2020-10-22 20:44:25.107 4169-4169/com.example.myapplication I/LOG_TAG: onCreate method !
2020-10-22 20:44:25.107 4169-4169/com.example.myapplication I/LOG_TAG: onCreateStandardActivity TaskId = 16 hashCode = 8507876
2020-10-22 20:44:25.108 4169-4169/com.example.myapplication I/LOG_TAG: taskAffinity : com.example.myapplication

2020-10-22 20:44:27.602 4169-4169/com.example.myapplication I/LOG_TAG: onCreate method !
2020-10-22 20:44:27.602 4169-4169/com.example.myapplication I/LOG_TAG: onCreateStandardActivity TaskId = 16 hashCode = 52597876
2020-10-22 20:44:27.603 4169-4169/com.example.myapplication I/LOG_TAG: taskAffinity : com.example.myapplication

2020-10-22 20:44:28.889 4169-4169/com.example.myapplication I/LOG_TAG: onCreate method !
2020-10-22 20:44:28.890 4169-4169/com.example.myapplication I/LOG_TAG: onCreateStandardActivity TaskId = 16 hashCode = 47979414
2020-10-22 20:44:28.891 4169-4169/com.example.myapplication I/LOG_TAG: taskAffinity : com.example.myapplication

2020-10-22 20:44:30.192 4169-4169/com.example.myapplication I/LOG_TAG: onCreate method !
2020-10-22 20:44:30.192 4169-4169/com.example.myapplication I/LOG_TAG: onCreateStandardActivity TaskId = 16 hashCode = 167910120
2020-10-22 20:44:30.193 4169-4169/com.example.myapplication I/LOG_TAG: taskAffinity : com.example.myapplication


```

可以看到，每个`standardActivity`对象的hashcode都不一样，说明是四个新的实例。同样的， 我们按四次返回键才能销毁这四个`Activity`。但是`TaskId`都一样的，因为我们是在同一个`Activity`里面启动了这个`standardActivity`，任务栈一样。



#### singleTop 栈顶复用模式

在这个启动模式下，如果当前这个`Activity`已经位于栈顶，那么它**不会被重新创建**，但是会调用它的`onNewIntent`方法。通过这个方法我们拿到当前的请求信息，进行处理。如果当前**任务栈不存在该Activity或者是不在栈顶**，则行为与standard模式相同了。



#### 