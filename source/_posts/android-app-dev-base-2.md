---
title: Android应用开发基础二：Activity
date: 2018-08-11 15:14:22
updated: 2022-01-10 17:52:48
tags:
- android
categories:
- Android开发
---

# Activity 生命周期

完整生命周期流程：

1. 一般的流程：onCreate() → onStart() → onResume() → onPause() → onStop() → onDestory()
2. 非正常销毁：onCreate() → onStart() → onResume() → onPause() → onCreate()
3. 非正常销毁：onCreate() → onStart() → onResume() → onPause() → onStop() →  onCreate()
4. 可见的返回：onCreate() → onStart() → onResume() → onPause() → onResume()
5. 不可见返回：onCreate() → onStart() → onResume() → onPause() → onStop() → onRestart() → onStart()

<!-- more -->

![android-activity-lifecycle](/images/android-activity-lifecycle.png)

**生命周期回调方法**

- onCreate()：执行基本启动逻辑。该逻辑在 Activity 的整个生命周期中只应发生一次，比如：声明界面、定义成员变量、配置界面、将数据绑定到列表，将 Activity 与 ViewModel 相关联、实例化某些类作用域变量等。
- onStart()：1. onStart() 调用后 Activity 对用户可见。2.初始化维护界面的代码。3. 回调会非常快速地完成。
- onResume()：可见并获得焦点；初始化在 onPause() 期间释放的组件。
- onPause()：1. 触发场景有常见的导航等打断事件、多窗口、半透明对话框等。2. 释放界面交互资源，如相机预览等。3. 回调会非常快速地完成，不能执行高负荷的任务。
- onStop()：1. 释放或调整在应用对用户不可见时的无用资源，例如，应用可以暂停动画效果，或从精确位置更新切换到粗略位置更新。2. 可以执行高负荷的任务，比如操作数据库等。3. 进入“已停止”状态后，对象将维护所有状态和成员信息，待恢复后，Activity 会重新调用这些信息。比如用户在 EditText 微件中输入文本，系统将保留文本内容，因此您无需保存和恢复文本。
- onDestory：释放先前的回调（例如 onStop()）尚未释放的所有资源。

注意

- 在onStart()中初始化的内容，需要在onStop里释放。
- 在onResume()初始化的内容，需要在onPause()里释放。
- 系统永远不会直接终止 Activity 以释放内存（直接kill），而是会终止 Activity 所在的进程。系统不仅会销毁 Activity，还会销毁在该进程中运行的所有其他内容。

# Activity 状态的变更

涉及用户和系统触发的事件导致 Activity状态的变更

## 配置发生了更改

- 场景：横竖屏切换、语言或输入设备的改变
- 配置发生更改时，Activity 会被销毁并重新创建。原始 Activity 实例将触发 onPause()、onStop() 和 onDestroy() 回调。系统将创建新的 Activity 实例，并触发 onCreate()、onStart() 和 onResume() 回调。
- 可使用 onSaveInstanceState() 方法和/或持久性本地存储，可使 Activity 的界面状态在配置发生更改后保持不变。

## Activity 或对话框显示在前台

当一个 Activity 启动另一个 Activity 时，它们都会经历生命周期转换。其原则为：首先自身失去焦点，然后把目标 Activity 尽快展示出来，最后在后台执行自身的 onStop()，以及 onDestroy()。

Activity A 进入 B :

1. Activity A 的 `onPause()` 方法执行。
2. Activity B 的 `onCreate()`、`onStart()`和 `onResume()`方法依次执行（Activity B 现在具有用户焦点）。
3. Activity A 不可见时，其 `onStop()`方法执行。

从 Activity B 返回 A ：

1. Activity B 的`onPause()` 方法执行。
2. Activity A 的 `onRestart()`、`onStart() `和 `onResume()`方法依次执行。
3. Activity B 的 `onStop()`方法执行。

## 用户点按“返回”按钮

如果 Activity 位于前台，并且用户点按了返回按钮，Activity 将依次经历 onPause()、onStop() 和 onDestroy() 回调。活动不仅会被销毁，还会从返回堆栈中移除。

需要注意的是，在这种情况下，默认不会触发 onSaveInstanceState() 回调。此行为基于的假设是，用户点按返回按钮时不期望返回 Activity 的同一实例。不过，您可以通过替换 onBackPressed() 方法实现某种自定义行为，例如“confirm-quit”对话框。

## 系统终止应用进程

如果某个应用处于后台并且系统需要为前台应用释放额外的内存，则系统可能会终止后台应用以释放更多内存。

# 状态保持和恢复

用户预期和系统行为：根据用户执行的操作，他们会希望系统清除或保留 Activity 状态。在某些情况下，系统会自动执行用户预期的操作。但有时，系统会执行与用户预期相反的操作。

状态保存：一般在onStop()之后，会触发 onSaveInstanceState() 回调，在此执行保存数据的操作。

状态恢复：在onCreate()和onRestoreInstanceState()回调中会传递 Bundle，恢复数据。

## 用户发起的清除界面场景

用户可以通过以下方式完全关闭 Activity：

- 从 Activity 向上导航
- 按返回按钮。不会触发 onSaveInstanceState() 回调
- 完成某种“完成”操作（由 Activity.finish() 提供支持）。不会触发 onSaveInstanceState() 回调
- 从“概览”（“最近使用的应用”）屏幕中滑动关闭 Activity
- 从“设置”中终止应用

> 💡 Android 是否会保存数据状态，主要依据用户的预期。主动销毁的 Activity 一般不会保存状态，也不会触发 onSaveInstanceState() 回调。

## 系统发起的清除界面场景

用户希望下次能恢复界面状态，但系统可能会清除的场景

- 配置更改。如横竖屏切换。
- 用户暂时切换到其他应用，稍后再返回到当前应用。在这种情况下，您的应用会被置于后台，系统会尽最大努力将您的应用进程留在内存中。但是，当用户转而去与其他应用进行互动时，系统可能会销毁该应用进程。在这种情况下，Activity 实例连同其中存储的任何状态都会一起被销毁。

这两种场景中，onSaveInstanceState()都会被调用，保存的数据在配置更改和进程终止后都会保留。

这里特别说下 配置更改的情况：

- 在配置更改时，系统自己会自动恢复界面控件的一些状态数据，比如EditText里的输入，但不会保存TextView等设置的数据，也不会保存 Activity 自定义的成员变量的数据。
- 可在 Android.Manifest 中设置 Activity 的属性，如：`android:configChanges="orientation|screenSize"`，使 Activity 在配置变化时不再销毁而重新创建实例，仅仅调用了`onConfigurationChanged`方法，可保持数据不变化。但官方并不建议这种自行处理配置变更的方法。

> 💡 注意：自行处理配置变更可能会提高使用备用资源的难度，因为系统不会为您自动应用这些资源。只有在必须避免 Activity 因配置变更而重启的无奈情况下，您才可考虑使用此方法，并且不建议对大多数应用使用此方法。

- 详细示例说明：状态改变及数据保存

  ```kotlin
  /* 
  经过横竖屏切换后，回调如下
  Application.onConfigurationChanged
  Activity.onPause()
  Activity.onStop()
  Activity.onSaveInstanceState
  Activity.onDestory()
  Activity.onCrate()
  Activity.onStart()
  Activity.onRestoreInstanceState()
  Activity.onResume()
  
  EditText里输入数据，横竖屏切换后不改变，保存了状态
  TextView btn_showTest 输入的4，横竖屏切换后重置，没有保存状态
  成员变量testInt只能通过onSaveInstanceState保存。
  
  ------
  在AndroidManifest文件，给 Activity 添加 android:configChanges="orientation|screenSize"
  再次切换横竖屏，回调如下
  Application.onConfigurationChanged
  Activity.onConfigurationChanged
  EditText、TextView、testInt状态都不变，都得到保存
  */
  class ActLearnActivity : AppCompatActivity() {
      companion object {
          private const val TAG = "ActLearnSample"
      }
  
      private var testInt = 0
  
      override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceState)
          setContentView(R.layout.activity_act_learn)
  				// testInt = 0
          Logger.d("testInt = $testInt")
  				// 切换前为 null， 切换后为4
          Logger.d("savedInstanceState testInt = " + savedInstanceState?.get("testInt")?.toString())
  				// 点击4次后切换横竖屏
          btn_changeTest.setOnClickListener {
              testInt = testInt.inc()
              btn_showTest.text = testInt.toString()
          }
      }
  
      override fun onRestoreInstanceState(savedInstanceState: Bundle) {
  				super.onRestoreInstanceState(savedInstanceState)
          Logger.d("testInt = " + savedInstanceState.get("testInt")?.toString())
      }
  
      override fun onSaveInstanceState(outState: Bundle) {
          super.onSaveInstanceState(outState)
          outState.putInt("testInt", testInt)
      }
  
  		override fun onConfigurationChanged(newConfig: Configuration) {
          super.onConfigurationChanged(newConfig)
          Logger.d("orientation = ${newConfig.orientation}")
      }
  }
  ```

## 保存并恢复数据的三种方法

您可以通过在各种类型的持久性机制之间划分工作，高效地保存和恢复界面状态。在大多数情况下，这些机制中的每一种都应存储 Activity 中使用的不同类型的数据，具体取决于数据复杂度、访问速度和生命周期的权衡：

- 本地持久性存储：存储在您打开和关闭 Activity 时不希望丢失的所有数据。示例：歌曲对象的集合，其中可能包括音频文件和元数据。
- `ViewModel`：在内存中存储显示关联界面控制器所需的所有数据。示例：最近搜索的歌曲对象和最近的搜索查询。
- `onSaveInstanceState()]`：存储当系统停止后又重新创建界面控制器时轻松重新加载 Activity 状态所需的少量数据。这里指的是将复杂的对象保留在本地存储空间中，并将这些对象的唯一 ID 存储在 `onSaveInstanceState()`中，而不是存储复杂的对象。示例：存储最近的搜索查询。

# 任务和回退栈

任务（Task）和回退栈（Back Stack）：任务是用户在执行某项工作时与之互动的一系列 Activity 的集合。这些 Activity 按照打开的顺序排列在一个回退栈中。

## 管理任务

一般情况，Activity 遵循”先进后出“的行为，可以通过设置\<activity> 清单元素或Intent改变这种行为。

> 💡 注意：大多数应用不应打破 Activity 和任务的默认行为。如果您确定需要让 Activity 改变默认行为，请谨慎操作，并且务必要测试该 Activity 在以下情况下的可用性：启动期间以及您通过返回按钮从其他 Activity 和任务返回该 Activity 时。务必要测试是否存在可能与用户预期的行为冲突的导航行为。

相关 `<activity>`属性包括：

- `taskAffinity`。亲和性，表示 Activity 在启动时倾向于添加到哪个回退栈中。默认情况下，同一应用中的所有 Activity 具有同一亲和性值。两种情况起作用：intent 包含 FLAG_ACTIVITY_NEW_TASK 标记时，以及Activity 的 allowTaskReparenting 属性设为 "true" 时。
- `launchMode`。启动模式。
- `allowTaskReparenting`。允许实例重新分配回退栈。
- `clearTaskOnLaunch`。
- `alwaysRetainTaskState`。
- `finishOnTaskLaunch`。

相关 intent 标记包括：

- `FLAG_ACTIVITY_NEW_TASK`。
- `FLAG_ACTIVITY_CLEAR_TOP`。
- `FLAG_ACTIVITY_SINGLE_TOP`。

## 启动模式及设置

设置启动模式，指定Activity应该如何关联到任务中。可通过清单或Intent 设置启动模式。

通过清单`launchMode` 属性可设置四种启动模式：

- "standard"（默认模式）。Activity 可以多次实例化，每个实例可以属于不同的任务，一个任务可以拥有多个实例。
- "singleTop"。基于"standard"，但回退栈顶部只能有一个实例。顶部没有实例时会创建实例，有实例时则调用 onNewIntent() 复用。
- "singleTask"。基于"singleTop"，但整个系统只有一个实例。没有实例时一般会创建新回退栈和实例，有实例时则调用 onNewIntent() 复用。
- "singleInstance"。基于"singleTask"，但实例所在的回退栈只有其一个 Activity实例而没有其他。没有实例时一般会创建新回退栈和实例，有实例时则调用 onNewIntent() 复用。

> 💡 从一个回退栈 A 启动在另一个回退栈 B 里的Activity实例后，连续返回则先把 B 里的Activity清除完后在清除A里的。

> 💡 onNewIntent()：当启动一个Activity，不是新建实例而是复用已有的实例时，已有实例会回调此方法。onNewIntent() 在 onStart() 后、onResume()前调用。

使用 Intent 标记，指定待启动 Activity 的模式：

- FLAG_ACTIVITY_NEW_TASK。与 "singleTask" launchMode 相同。
- FLAG_ACTIVITY_SINGLE_TOP。与 "singleTop" launchMode 相同。
- FLAG_ACTIVITY_CLEAR_TOP。如果要启动的 Activity 已经在当前任务中运行，则不会启动该 Activity 的新实例，而是会销毁位于它之上的所有其他 Activity，并通过 onNewIntent() 将此 intent 传送给它的已恢复实例（现在位于堆栈顶部）。FLAG_ACTIVITY_CLEAR_TOP 最常与 FLAG_ACTIVITY_NEW_TASK 结合使用。

**注意：清单和代码冲突时，以 Intent 标记为准**。

# 参考

- 官网-[了解 Activity 生命周期](https://developer.android.google.cn/guide/components/activities/activity-lifecycle?hl=zh-cn#java)
- [官网-了解任务和返回堆栈](https://developer.android.google.cn/guide/components/activities/tasks-and-back-stack?hl=zh-cn)
- [保存界面状态](https://developer.android.google.cn/topic/libraries/architecture/saving-states#ui-dismissal-system)
- [处理配置变更](https://developer.android.google.cn/guide/topics/resources/runtime-changes#HandlingTheChange)
