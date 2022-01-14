---
title: 探讨-如何在子线程直接刷新主线程的UI
date: 2021-11-07 15:18:11
tags:
- android
- android系统开发
categories:
- Android开发
---

本文主要探讨如何在子线程中不通过Handler而直接更新主线程的UI的问题，仅探讨。

<!-- more -->

# 问题场景

之前被问到一个问题：如何在子线程中不通过Handler而直接更新主线程的UI，比如在主线程创建的TextView，如何在子线程直接调用`setText()`方法更新文本？实例代码如下：

```java
public class MainActivity extends AppCompatActivity {
    private TextView tv1;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        tv1 = findViewById(R.id.tv_hw);
        tv1.setOnClickListener(v -> new Thread(() -> {
            tv1.setText("子线程刷新UI");
        }).start());
    }
}
```

直接运行上述代码会导致程序崩溃，调用栈如下：

```bash
E/AndroidRuntime: FATAL EXCEPTION: Thread-2
    Process: com.xxx.viewsettextdemo, PID: 4387
    android.view.ViewRootImpl$CalledFromWrongThreadException: Only the original thread that created a view hierarchy can touch its views.
        at android.view.ViewRootImpl.checkThread(ViewRootImpl.java:8191)
        at android.view.ViewRootImpl.requestLayout(ViewRootImpl.java:1420)
        at android.view.View.requestLayout(View.java:24469)
        at android.view.View.requestLayout(View.java:24469)
        at android.view.View.requestLayout(View.java:24469)
        at android.view.View.requestLayout(View.java:24469)
        at android.view.View.requestLayout(View.java:24469)
        at android.view.View.requestLayout(View.java:24469)
        at androidx.constraintlayout.widget.ConstraintLayout.requestLayout(ConstraintLayout.java:3146)
        at android.view.View.requestLayout(View.java:24469)
        at android.widget.TextView.checkForRelayout(TextView.java:9681)
        at android.widget.TextView.setText(TextView.java:6269)
        at android.widget.TextView.setText(TextView.java:6097)
        at android.widget.TextView.setText(TextView.java:6049)
        at com.xxx.viewsettextdemo.MainActivity.lambda$onCreate$0$MainActivity(MainActivity.java:22)
        at com.xxx.viewsettextdemo.-$$Lambda$MainActivity$_uRJsNnQ0-zyKYjBJhwoyKb6E_I.run(Unknown Source:2)
        at java.lang.Thread.run(Thread.java:919)
```

有什么方法，保证程序不崩溃且能够更新UI？

# 探讨

问题出现在调用`ViewRootImpl.checkThread()`方法时，抛出错误："CalledFromWrongThreadException: Only the original thread that created a view hierarchy can touch its views."

```java
void checkThread() {
    if (mThread != Thread.currentThread()) {
        throw new CalledFromWrongThreadException(
                "Only the original thread that created a view hierarchy can touch its views.");
    }
}
```

异常消息是说，只有创建View所属视图树的线程才能更新该View，判断依据是ViewRootImpl的mThread成员与当前线程不一致。mThread只在ViewRootImpl构造方法里进行赋值，而ViewRootImpl是在WindowManager(实际为WindowManagerGlobal)的addView方法里进行实例化的，且不能被应用开发者直接实例化。在不更改系统代码的情况下，可以进行如下尝试：

### 方案 1

在主线程中移除目标View，然后在子线程通过WindowManager.addView()添加该view。addView()会创建新的ViewRootImpl实例，并将目标View添加新的ViewRootImpl对应的视图树中。这种方法仅仅解决了在子线程中更新主线程中创建的View问题，但在UI层面则表现为子线程创建了一个新的窗口，此时目标View已经离开了主线程创建的窗口而位于新窗口中，更新的也仅仅是新窗口中的UI，并不能算是子线程更新了主线程的UI，此时主线程UI会缺失目标View。

具体代码如下，在上述代码中，添加一个按钮，点击按钮时触发创建子线程操作：

```java
btn.setOnClickListener(v -> new Thread(() -> {
    // 主线程中的View只能在主线程中移除
    MainActivity.this.runOnUiThread(() -> {
        ViewGroup parent = (ViewGroup) tv1.getParent();
        parent.removeView(tv1);
    });
    // addView()时会调用Handler发消息到该线程，需要Looper
    Looper.prepare();
    WindowManager wm = getWindowManager();
    WindowManager.LayoutParams lp = new WindowManager.LayoutParams(
        WindowManager.LayoutParams.MATCH_PARENT,
        WindowManager.LayoutParams.WRAP_CONTENT,
        WindowManager.LayoutParams.TYPE_APPLICATION,
        WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE
        | WindowManager.LayoutParams.FLAG_NOT_TOUCH_MODAL,
        PixelFormat.TRANSLUCENT);
    wm.addView(tv1, lp);
    tv1.setTextColor(Color.BLACK);
    tv1.setText("我是在子线程中的的更新的UI");
    Looper.loop();
}).start());
```

具体效果如下

进入应用，依次为TextView和Button：

![](/images/android-update-ui-2.png)

点击“子线程更新UI”的Button，TextView出现在新的窗口中且文本被更新：

![](/images/android-update-ui-1.png)

上述方案与直接在子线程中创建View实例是一样的，唯一的区别是创建实例的地方不一样。该方案仅仅解决了如何在子线程中更新主线程创建的View实例信息这个问题而已，并没有完全解决文章开始提到的问题。

不修改系统代码，而仅仅使用目前的SDK提供的API，我认为很难解决该问题，因为无法绕过`ViewRootImpl.checkThread()`方法，也许有其他的Trick，待以后跟进。

