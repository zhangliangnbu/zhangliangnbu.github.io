---
title: GridLayoutManager实现均匀分布
date: 2018-11-04 11:11:16
tags:
- android
- GridLayoutManager
categories:
- Android开发
---

# 背景描述
最近开发中遇到需求，需要实现图片的格子分布效果，如下图所示

![格子图片](https://upload-images.jianshu.io/upload_images/2658578-22632ab28c5aedd6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中要求：

1. 图片与图片的间隔，图片与屏幕的左边距，以及图片与屏幕的右边距，都为固定大小，比如`10dp`。
2. 图片宽高相等。

-- -- --

<br>

# 代码实现

主要代码如下
```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        val images = listOf(...)
        val imageAdapter = ImageAdapter(images)
        rv_images.apply {
            layoutManager = GridLayoutManager(context, 4, RecyclerView.VERTICAL, false)
            addItemDecoration(EvenItemDecoration(dp2px(context, 10), 4))
            adapter = imageAdapter
        }
    }

    class ImageAdapter(private val images: List<Int> = listOf()) : RecyclerView.Adapter<ImageAdapter.ViewHolder>() {
        override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
            val view = LayoutInflater.from(parent.context).inflate(R.layout.item_image, parent, false)
            // 动态设置图片大小 保证宽高相等
            val ivSize = (getScreenContentWidth(parent.context) - dp2px(parent.context, 10) * 5) / 4
            view.iv_image.layoutParams.height = ivSize
            return ViewHolder(view)
        }

        override fun onBindViewHolder(holder: ViewHolder, position: Int) {
            holder.view.iv_image.setImageResource(images[position])
        }
        ...
    }


    class EvenItemDecoration(private val space: Int, private val column: Int) : RecyclerView.ItemDecoration() {
        override fun getItemOffsets(outRect: Rect, view: View, parent: RecyclerView, state: RecyclerView.State) {
            val position = parent.getChildAdapterPosition(view)
            // 每列分配的间隙大小，包括左间隙和右间隙
            val colPadding = space * (column + 1) / column
            // 列索引
            val colIndex = position % column
            // 列左、右空隙。右间隙=space-左间隙
            outRect.left = space * (colIndex + 1) - colPadding * colIndex
            outRect.right = colPadding * (colIndex + 1) - space * (colIndex + 1)
            // 行间距
            if (position >= column) {
                outRect.top = space
            }
        }
    }}
```
布局`item_image.xml`如下：
```xml
<?xml version="1.0" encoding="utf-8"?>
<ImageView
    android:id="@+id/iv_image"
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:scaleType="fitXY"
    tools:src="@mipmap/ic_launcher"
    android:background="#ffccff"/>
```
其中需要注意是
1. 图片的宽度在布局和代码中不能写死，需要自适应。
2. 在代码（Adapter）中根据屏幕宽度来计算图片高度。
3. 在ItemDecoration中计算每列的左右间隙。

-- -- --

<br>

# 分析-GridLayoutManger分配Item空间的机制
**GridLayoutManger在绘制子View的时候，会先为它们分配固定的空间**。
比如，我们这里是4列，则每列分配父布局宽度（这里是屏幕宽度） / 4的宽度大小的空间，每列占据的宽度相等。

1. 每列由图片和空隙组成。空隙包括左空隙和右空隙，分布在图片的左右。
2. 每列中的图片会局限在列分配的空间内，不会超出这个空间。如果图片宽度小于列宽，则剩余的宽度会分配给空隙；如果图片的宽度大于列宽，则不会有空隙，且图片超出列的部分会被遮盖。
3. 如果使用ItemDecoration设置间隙，则间隙不会被压缩，如果图片过宽，则会占用当前图片的空间，图片会被压缩。

**示例一-设置固定大宽度**
在Adapter代码中设置图片的宽度并在去掉ItemDecoration

```kotlin
// ImageAdapter
...
val ivSize = (getScreenContentWidth(parent.context) - dp2px(parent.context, 10) * 5) / 4
view.iv_image.layoutParams.width = 1000
view.iv_image.layoutParams.height = ivSize
...

// MainActivity 中注释掉EvenItemDecoration
...
// addItemDecoration(EvenItemDecoration(dp2px(context, 10), 4))
...
```
结果如下：

![宽度设置为1000px的图片](https://upload-images.jianshu.io/upload_images/2658578-0d38e5c7cf5fd001.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过Studio自带`Layout Inspector`可以查看每个图片的左右位置，第一行图片的`mLeft`分为0、360、720、1080，对应的`mRight`是`mLeft` + 1000。
以上示例说明

1. 每个图片占据的父布局的宽度是一样的，为父布局宽度除以列数，这里是1440 / 4 = 360。
2. 每个图片超出所占据空间宽度后的部分会被后面的图片或屏幕遮盖。

**示例二-设置较大的左右间隔**
修改EvenItemDecoration代码如下：

```kotlin
// MainActivity中，设置间隙为20dp
        rv_items.apply {
            layoutManager = GridLayoutManager(context, 4, RecyclerView.VERTICAL, false)
            addItemDecoration(EvenItemDecoration(dp2px(context, 20), 4))
            adapter = imageAdapter
        }

// 修改EvenItemDecoration
	class EvenItemDecoration(private val space: Int, private val column: Int) : RecyclerView.ItemDecoration() {
        override fun getItemOffsets(outRect: Rect, view: View, parent: RecyclerView, state: RecyclerView.State) {
            outRect.left = space
            outRect.right = space
        }
    }
```
运行结果如下：

![有间隔的Grid](/images/gridlayout_even_example2.png)

通过`Layout Inspector`查看每个图片的左右位置，每个图片的大小为220，左右间隙都为70(px/dp=3.5)，说明设置的间隙会占用图片之前分配的空间。

最后，根据以上机制，不难理解`EvenItemDecoration`中的代码。

Github源代码地址:https://github.com/zhangliangnbu/gridlayout-demo
