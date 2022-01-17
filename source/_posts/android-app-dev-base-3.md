---
title: Android应用开发基础三：Fragment
date: 2018-08-12 15:14:22
updated: 2022-01-10 17:52:48
tags:
- android
categories:
- Android开发
---

# 概述

Fragment 表示应用界面中可重复使用的一部分，不能独立存在，而是必须由 Activity 或另一个 Fragment 托管。

模块化和可重用性：Fragment之间需要解耦，避免让一个 Fragment 依赖于另一个 Fragment 或从一个 Fragment 操控另一个 Fragment。

添加 Fragment 到 Activity 中的两种方法：

- 直接在布局里定义fragment元素。
- 在布局定义 fragment 容器，然后在代码中将 fragment 添加到容器中。推荐使用 AndroidX 的FragmentContainerView。此控件中也可以直接定义 Fragment 进行初始化。

Fragment 创建时处于 INITIALIZED 状态，之后需要添加到 FragmentManager 才能转移到下一个状态。Fragment 的状态转移由 FragmentManger 通过 FragmentTransaction 来管理。

<!-- more -->

# 生命周期

生命周期回调方法都发生在 onAttach() 和 onDetach() 之间，图中未标记。

<img src="/images/android-fragment-lifecycle.png" width="70%" height="70%">

与Activity声明周期对应关系

```bash
onAttach()：和宿主Activity建立关联时调用
onCreate()：创建时调用
onCreateView()：加载Fragment的布局时调用
onActivityCreated()：宿主Activity创建完毕时调用
onStart()：可见时调用
onResume()：获取焦点时用
onPause()：失去焦点时调用
onStop()：不可见时调用
onDestoryView()：移除Fragment的布局时调用
onDestory()：销毁时调用
onDetach()：与宿主Activity解除关联时调用
```

# FragmentManager & FragmentTransaction

FragmentManager 中有两个数据集合：一个是存储 Fragment 的集合，一个是存储事务 Transaction 的回退栈（BackStack）。

两种对象的操作：

- Fragment 的操作通过 FragmentTransaction 的 add/replace/remove来管理。
- Transaction 的操作通过 FragmentTransaction#addToBackStack 和 FragmentManager#popBackStack 来管理。Transaction 的返回操作可以通过按手机返回键回调。

```java
// 示例：
// 连续添加两个Fragment，只添加一个Transaction，回退栈 count = 1
// 如果连续remove Fragment 则当前Activity不再显示任何C和D, 而回退栈 count = 1
// 此时按返回键，则显示C和D进入销毁和Detach状态。
// 再次按返回键，则回退到上一个Activity
// 按返回键会调用 FragmentManager#popBackStack 
supportFragmentManager.beginTransaction()
    .add(R.id.fl_container, SampleCFragment(), "C")
    .add(R.id.fl_container, SampleDFragment(), "D")
    .addToBackStack("CD")
    .commit()
```

FragmentTransaction 方法

- add。添加一个新的 Fragment
- remove。移除一个Fragment
- replacement。先remove 容器里所有的Fragment，再添加一个新的 Fragment。
- hide / show。前者隐藏一个Fragment，后者将隐藏的Fragment展示出来。不走方法回调。
- detach：View 结构被销毁，处于STOPPED状态，但依旧保存在FragmentManager的集合中 。
- attach：重新创建并展示。

> 💡 FragmentTransaction 的 detach() 和 attach() 方法与 Fragment 的 onAttach() 和 Attach() 无关。

# Fragment 之间添加动画过度效果

方法一，动画框架（Animation Framework）

```java
// 定义动画
...
// 设置动画
Fragment fragment = new FragmentB();
getSupportFragmentManager().beginTransaction()
    .setCustomAnimations(
        R.anim.slide_in,  // enter
        R.anim.fade_out,  // exit
        R.anim.fade_in,   // popEnter
        R.anim.slide_out  // popExit
    )
    .replace(R.id.fragment_container, fragment)
    .addToBackStack(null)
    .commit();
```

方法二，转场框架（Transition Framework）

```java
public class FragmentA extends Fragment {
    @Override
    public View onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        TransitionInflater inflater = TransitionInflater.from(requireContext());
        setExitTransition(inflater.inflateTransition(R.transition.fade));
    }
}
public class FragmentB extends Fragment {
    @Override
    public View onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        TransitionInflater inflater = TransitionInflater.from(requireContext());
        setEnterTransition(inflater.inflateTransition(R.transition.slide_right));
    }
}
```

# Fragment 通信

ViewModel 和 Fragment Result API 。

**ViewModel**：在多个 Fragment 之间或 Fragment 与其宿主 Activity 之间共享数据。

```java
// 示例
// 在 Activity 里 ...
viewModel = new ViewModelProvider(this,
                new ViewModelProvider.AndroidViewModelFactory(getApplication()))
          .get(ItemViewModel.class);
viewModel.getSelectedItem().observe(this, new Observer<ItemViewModel.Item>() {
    @Override
    public void onChanged(ItemViewModel.Item item) {
        tvTitle.setText(item.content);
    }
});

// 在Fragment 里 ...
viewModel = new ViewModelProvider(requireActivity(),
                ViewModelProvider.AndroidViewModelFactory.getInstance(getActivity().getApplication()))
                .get(ItemViewModel.class);

OnItemClickListener listener = new OnItemClickListener() {
    @Override
    public void onClick(ItemViewModel.Item item) {
        viewModel.selectItem(item);
    }
};
```

**使用 Fragment Result API**

- 要求：`androidx.fragment:fragment:1.3.0` 之后。
- 使用`FragmentManager#setFragmentResultListener` 和 `FragmentManager#setFragmentResult`

```java
// 示例
// Activity 中
getSupportFragmentManager().setFragmentResultListener("msgKey", this, new FragmentResultListener() {
    @Override
    public void onFragmentResult(@NonNull @NotNull String requestKey, @NonNull @NotNull Bundle result) {
        String results = result.getString("bundleKey");
        tvTitleByApi.setText(results);
    }
});

// Fragment 中
btn_send_msg.setOnClickListener {
    val result = Bundle()
    result.putString("bundleKey", "result 来啦 haha")
    parentFragmentManager.setFragmentResult("msgKey", result)
}
```

# Fragment 懒加载实现

普通方式

- `add+show+hide` 模式。利用 `onHiddenChanged()`回调方法，以及`isHidden()` 判断；在 `onResume()` 和 `onHiddenChanged()`里编写加载数据的代码 。
- `ViewPager+Fragment`模式。利用 `setUserVisibleHint()`回调方法。

缺点：onResume都会被调用

Androidx 下的懒加载

- 利用 `FragmentTransaction#setMaxLifecycle` 方法，强行限制Fragment所能达到的最大状态。

缺点：Fragment 的嵌套时，失效。

# 参考

- [Android 官方开发文档](https://developer.android.google.cn/guide/fragments?hl=zh-cn)
- [Androidx 下 Fragment 懒加载的新实现](https://juejin.cn/post/6844904050698223624)
