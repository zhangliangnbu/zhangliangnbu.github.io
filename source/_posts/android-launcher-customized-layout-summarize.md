---
title: Launcher开发总结
date: 2020-12-27 15:10:09
tags:
- android
categories:
- Android开发
---

之前参与了Launcher项目，负责样式的自定义布局部分。这部分主要涉及到的技术点是自定义布局和动画，难度适中，但略繁杂，其中有几个技术相关的难点记录如下。

# 介绍

布局基于二维格子：将图标显示区划分为`m x n`个格子，每个图标占据一个或多个格子。自由布局样式里，格子划分得更小，每个图标的最小尺寸有规定，不能小于规定值。

图标大小可更改：不同尺寸的图标展示的信息不一样，最小的图标仅展示一个ICON，最大的图标可以展示应用丰富的信息以及操作。展示的信息是通过与应用提供的服务组件或广播来通信的。

# 手指长按触发拖动

下图边框表示一个ViewGroup，包含8个子View。我们希望长按某个子View，并拖动它到其他的位置，比如长按View-1后，显示能够拖动，然后拖动它到View-7的位置。

![android-launcher-customized-1.drawio](/images/android-launcher-customized-1.drawio.svg)

Github上应该有相关框架，可以自己实现但不需要，因为Android提供了相关API，见[官方文档](https://developer.android.com/guide/topics/ui/drag-drop)。其原理是创建待拖动View的副本，并跟随手指触摸坐标更改副本View的位置。使用流程为在View的长按监听回调`OnLongClickListener.onLongClick(View v)`里调用开启拖动的方法`View.startDragAndDrop()`，然后在目标容器中注册拖动回调监听即可触发拖动事件的回调方法`OnDragListener.onDragEvent()`，之后根据坐标判断拖动结束的地方或View。

<!-- more -->

关键代码如下：

1. 定义拖拽监听，处理drop和end(如果需要)事件

   ```java
   // 拖拽监听
   public class ItemDragEventListener implements View.OnDragListener {
       public void addEventSender(ViewGroup cell) {
           cell.setOnDragListener(this);
       }
   
       public boolean onDrag(View v, DragEvent event) {
           final int action = event.getAction();
           switch(action) {
               case DragEvent.ACTION_DRAG_STARTED:
               case DragEvent.ACTION_DRAG_ENTERED:
               case DragEvent.ACTION_DRAG_LOCATION:
               case DragEvent.ACTION_DRAG_EXITED:
                   return true;
               case DragEvent.ACTION_DROP:
                   return onDrop(v, event);
               case DragEvent.ACTION_DRAG_ENDED:
                   return onEnd(v, event);
               default:
                   break;
           }
           return false;
       }
   
       // 处理drop事件
       private boolean onDrop(View v, DragEvent event) {
           // from info
           View view = (View) event.getLocalState();
           ViewGroup fromCell = (ViewGroup) view.getParent();
           int fromIndex = fromCell.indexOfChild(view);
   
           // current(target) info
           final ViewGroup targetCell = (ViewGroup) v;
           int targetIndex = Utils.getChildIndexStrictly(targetCell, event.getX(), event.getY());
   
           // move item
           if (targetIndex >= 0 && targetIndex < targetCell.getChildCount()) {
               moveItem(fromCell, fromIndex, targetCell, targetIndex);
           }
           return true;
       }
   }
   ```

   其中，通过坐标定位目标图标View的方法如下：

   ```java
   public static int getChildIndexStrictly(ViewGroup fl, float x, float y) {
       int count = fl.getChildCount();
       int left, top, right, bottom;
       for (int i = 0; i < count; i ++) {
           final View child = fl.getChildAt(i);
           left = child.getLeft();
           right = child.getRight();
           top = child.getTop();
           bottom = child.getBottom();
           if (x >= left && x <= right && y >= top && y <= bottom) {
               return i;
           }
       }
       return -1;
   }
   ```

2. 将拖拽监听设置给目标容器

   ```java
   itemDragEventListener.addEventSender(viewGroup);
   ```

3. 在待拖拽的图标View长按监听里设置拖拽事件

   ```java
   // 长按监听
   public static class ItemLongClickListener implements OnLongClickListener {
       @Override
       public boolean onLongClick(View v) {
           v.startDragAndDrop(null, new DragShadowBuilder(aiv), aiv, 0);
           return true;
       }
   }
   // 设置长按监听
   itemView.setOnLongClickListener(new ItemLongClickListener());
   ```



# 拖动View到页面边界触延时连续页面切换

在一个ViewPager里，我们希望拖动当前Pager里的一个View到Pager的边界时，会自动切换Pager。但切换之后我们不希望马上连续切换，因为我们希望在切换后的Pager里，留给用户些许时间来判断是否释放掉拖动的View。比如下图中，我们拖动View-2到Pager的左边界时，我们希望它切换到第2个页面，并且留些许时间让用户考虑是否将View-2放在页面2，如果不放在页面2，用户会继续保持长按，则会继续切换到页面1。

![android-launcher-customized-2](/images/android-launcher-customized-2.svg)

实现：监听`DragEvent.ACTION_DRAG_LOCATION`事件，获取实时坐标，判断是否需要切换Pager，如果切换则发送一个延时消息，在这个延时消息触发之前不允许再次切换Pager。

关键代码如下：

```java
// 监听DragEvent.ACTION_DRAG_LOCATION事件
public boolean onDrag(View v, DragEvent event) {
    final int action = event.getAction();
    switch(action) {
        case DragEvent.ACTION_DRAG_LOCATION:
            return onLocation(v, event);
    }
    return false;
}

private boolean onLocation(View v, DragEvent event) {
    // 如果当前已经触发了切换Pager的动作，则不能马上继续切换，需要等待isScrolling置为false
    if (isScrolling) {
        return true;
    }
    // scrollWidthPx为触发切换Pager的区域宽度，比如为150px
    boolean isScrollToLeft = x < scrollWidthPx && currentIndex > 0;
    boolean isScrollToRight = x > w - scrollWidthPx && currentIndex < lastIndex;
    if (isScrollToLeft || isScrollToRight) {
        vp.setCurrentItem(isScrollToLeft ? currentIndex - 1 : currentIndex + 1, true);
        isScrolling = true;
        // 延时scrollDelayedTime（可设置为1s）后设置isScrolling=false
        handler.sendEmptyMessageDelayed(DelayedScrollHandler.MSG_SCROLL_END, scrollDelayedTime);
    }
    return true;
}

// 伪代码，实际中需使用弱引用
public static class DelayedScrollHandler extends Handler {
    public static final int MSG_SCROLL_END = 1001;
    @Override
    public void handleMessage(@NonNull Message msg) {
        if (msg.what == MSG_SCROLL_END) {
            isScrolling = false;
        }
    }
}
```



# 缩放父视图而不改变子视图的动画

实现一个视图的缩放动画是简单的，默认会缩放子视图。如果我们仅仅只缩放俯视图而要保持子视图的尺寸，则如何实现呢？有两种实现方法：

1. 利用属性动画ValueAnimator改变缩放值，在`AnimatorUpdateListener.onAnimationUpdate()`回调里实时更新父视图的MarginLayoutParams以及子视图的LayoutParams。这个方法在计算奇数除2时会导致1像素的损失而产生抖动现象，可以考虑使用浮点数，待以后验证 **TODO**。
2. 同时利用属性动画ObjectAnimator对父视图和子视图同时缩放，但方向相反，如此子视图的尺寸能得到保持，动画流畅。

两种方法的关键代码如下

方法一：利用属性动画ValueAnimator改变缩放值，存在画面抖动现象

```java
private Animator getKeepChildScaleAnimator(View from, final View target) {
    // 初始化参数...
    // 数值动画
    ValueAnimator va = ValueAnimator.ofPropertyValuesHolder(widthProperty, heightProperty);
    va.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
        @Override
        public void onAnimationUpdate(ValueAnimator animation) {
            // 更新父视图尺寸
            int width = (int) animation.getAnimatedValue(PROPERTY_SCALE_WIDTH);
            int height = (int) animation.getAnimatedValue(PROPERTY_SCALE_HEIGHT);
            // todo 差为奇数时存在抖动 考虑使用float
            int deltaMarginTop = (targetHeight - height) / 2;
            int deltaMarginLeft = (targetWidth - width) / 2;
            ViewGroup.MarginLayoutParams lp = (ViewGroup.MarginLayoutParams) target.getLayoutParams();
            ...
            // 更新子视图尺寸
            ViewGroup vp = (ViewGroup) target;
            View child = vp.getChildAt(0);
            ViewGroup.LayoutParams childLp = child.getLayoutParams();
            childLp.width = width - target.getPaddingLeft() - target.getPaddingRight();
            childLp.height = height - target.getPaddingTop() - target.getPaddingBottom();
        }
    });
    return va;
}
```

方法二：同时利用属性动画ObjectAnimator对父视图和子视图同时缩放

```java
public Animator getKeepChildScaleAnimator(View from, final View target) {
    // parent
    final float scaleX = 1.0f * from.getWidth() / target.getWidth();
    final float scaleY = 1.0f * from.getHeight() / target.getHeight();
    PropertyValuesHolder scaleXProperty = PropertyValuesHolder.ofFloat("scaleX", scaleX, 1);
    PropertyValuesHolder scaleYProperty = PropertyValuesHolder.ofFloat("scaleY", scaleY, 1);
    ValueAnimator va = ObjectAnimator.ofPropertyValuesHolder(target, scaleXProperty, scaleYProperty);
    if (! (target instanceof ViewGroup)) {
        return va;
    }

    // child
    final float childScaleX = 1.0f / scaleX;
    final float childScaleY = 1.0f / scaleY;
    PropertyValuesHolder childScaleXProperty = PropertyValuesHolder.ofFloat("scaleX", childScaleX, 1);
    PropertyValuesHolder childScaleYProperty = PropertyValuesHolder.ofFloat("scaleY", childScaleY, 1);
    ValueAnimator childVa = ObjectAnimator.ofPropertyValuesHolder(((ViewGroup) target).getChildAt(0), childScaleXProperty, childScaleYProperty);
    AnimatorSet set = new AnimatorSet();
    set.playTogether(va, childVa);
    return set;
}
```

# ViewPager页面间拖动子视图更改位置的动画

我们希望能在ViewPager页面间拖动并改变页面里的View的位置，比如下图中，我们希望拖动当前Pager3中View-2到Pager2的View-2位置上，那么Pager2中View-2就会被挤到View-3的位置，View-3会被挤到Pager3中的View-1的位置，我们希望View-X的位置改变都以动画展示出来，其他动画比较容易实现，难点是如何在Pager2页面展示View-3被挤到Pager3中的动画，即View-3向右离开Pager2的动画。

![android-launcher-customized-2](/images/android-launcher-customized-2.svg)

实现：由于最终的Pager2视图中是不会出现View-3的，因此需要使用一个额外的视图来表现View-3离开Pager2页面的动画，并在动画结束时移除View。

具体的方法也有两种：

1. 在应用级的Window里新增View并展示动画，最后发现动画会闪烁。待调查动画闪烁原因 **TODO**
2. 直接在Pager里添加View，并在动画结束时移除View。动画流程。

关键代码如下

方法一：使用窗口展示动画

```java
// 动画有闪烁，不用
private void moveSingleByAdditionWindow(View target, IItemInfo squeezeItemInfo, int horizontalIndexDistance) {
    // 创建窗口视图
    final View virtual = createVirtualView(squeezeItemInfo);
    WindowManager wm = (WindowManager) target.getContext().getSystemService(Context.WINDOW_SERVICE);
    WindowManager.LayoutParams lp = new WindowManager.LayoutParams();
    // 设置LayoutParams...略
    wm.addView(virtual, lp);
	
    // 通过ValueAnimator更新窗口LayoutParams
    int dx = horizontalIndexDistance * target.getWidth();
    ValueAnimator va = ValueAnimator.ofInt(lp.x, lp.x + dx);
    va.addUpdateListener(animation -> {
        lp.x = (int) animation.getAnimatedValue();
        wm.updateViewLayout(virtual, lp);
    });

    // 动画结束时移除窗口视图
    va.addListener(new AnimatorListenerAdapter() {
        @Override
        public void onAnimationEnd(Animator animation) {
            wm.removeView(virtual);
        }
    });

    // 设置透明动画
    ObjectAnimator alphaAnim = ObjectAnimator.ofFloat(virtual, "alpha", child.getAlpha(), 0f);
    startSet(va, alphaAnim);
}
```

方法二：直接在Pager里添加View

```java
// 在父容器里直接添加view，动画完后再删除，动画效果流畅
private void moveVirtualSingleByAddRemove(View target, IItemInfo squeezeItemInfo, int horizontalIndexDistance) {
    // 创建View，并添加到父布局里
    final View virtualView = createVirtualView(...);
    // 动画
    int dx = horizontalIndexDistance * target.getWidth();
    ObjectAnimator translationXAnim = ObjectAnimator.ofFloat(virtualView, "translationX", -dx, 0f);
    // 移除View
    translationXAnim.addListener(new AnimatorListenerAdapter() {
        @Override
        public void onAnimationEnd(Animator animation) {
            removeVirtualView(...)
        }
    });
    // 透明动画
    ObjectAnimator alphaAnim = ObjectAnimator.ofFloat(virtualView, "alpha", child.getAlpha(), 0f);
    startSet(translationXAnim, alphaAnim);
}
```

# 参考

- [官方文档-Drag And Drop](https://developer.android.com/guide/topics/ui/drag-drop)
