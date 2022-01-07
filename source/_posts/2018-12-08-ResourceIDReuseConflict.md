---
title: EditText的ID相同而导致的冲突
date: 2018-12-08 17:00:03
tags:
- android
- id
categories:
- Android开发
---

# 问题描述

最近写项目时，遇到两个自定义ViewGroup中EditText的ID相同而导致冲突的问题，描述如下。
1.  我自定义了一个继承FrameLayout名为ClearableEditText的控件，里面包括一个AutoCompleteTextView(EditText的一种) 和一个ImageView。
2.  我在LoginFragment（继承Fragment）里使用了两个ClearableEditText，并输入了值分别为123456789和abc。
3.  我从LoginFragment里使用FragmentTransaction.replace方法跳转到另外一个Fragment，当回退到LoginFragment时，两个ClearableEditText都显示了abc。

<!-- more -->

相关代码如下：

自定义的ClearableEditText
```kotlin
class ClearableEditText @JvmOverloads constructor(context: Context, attrs: AttributeSet? = null, defStyleAttr: Int = 0, defStyleRes: Int = 0)
    : FrameLayout(context, attrs, defStyleAttr, defStyleRes) {

    init {
        initView(context, attrs, defStyleAttr, defStyleRes)
    }

    lateinit var input: AutoCompleteTextView
    private lateinit var clearImage: ImageView

    private fun initView(context: Context, attrs: AttributeSet?, defStyleAttr: Int, defStyleRes: Int) {
        // init view
        val view = LayoutInflater.from(context).inflate(R.layout.view_clearable_edit_text, this, true)
        input = view.findViewById<AutoCompleteTextView>(R.id.et_clearable)
        clearImage = view.findViewById(R.id.iv_clearable)

        // init attr ......
    }

    override fun onSaveInstanceState(): Parcelable {
        Logger.d("view life -> onSaveInstanceState")
        return super.onSaveInstanceState()
    }

    override fun onRestoreInstanceState(state: Parcelable?) {
        Logger.d("view life -> onRestoreInstanceState")
        super.onRestoreInstanceState(state)
    }
}
```
view_clearable_edit_text布局
```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tool="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    >

    <AutoCompleteTextView
        android:id="@+id/et_clearable"
        style="@style/AuthRowValue"
        android:layout_width="match_parent"
        android:layout_marginTop="@dimen/spacing_row_vertical"
        tool:drawableStart="@drawable/ic_mobile"
        tool:ignore="LabelFor"/>

    <ImageView
        android:id="@+id/iv_clearable"
        style="@style/MarkDelete"
        android:visibility="gone"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        tool:ignore="ContentDescription"
        tool:visibility="visible"/>
</android.support.constraint.ConstraintLayout>
```


LoginFragment
```kotlin
class LoginFragment : BaseFragment() {
    override fun getFragmentID(): FragmentID {
        return FragmentID.LOGIN
    }

    companion object {
        val TAG = this::class.java.simpleName!!
    }

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        Logger.d("fragment life circle---->onCreateView")
        return inflater.inflate(R.layout.fragment_login, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
   
        // 注册
        tv_register.setOnClickListener { toRegister() }

        // .....
    }

    private fun toRegister() {
        val ba = activity as BaseActivity
        ba.navigateReplace(RegistrationFragment(), getString(R.string.auth_register), true)
    }
}
```

fragment_login 布局
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tool="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/background_page_dark"
    android:paddingEnd="@dimen/spacing_24"
    android:paddingStart="@dimen/spacing_24"
    android:orientation="vertical"
    >
    <include layout="@layout/view_logo"/>

    <cn.yintech.cdam.view.ClearableEditText
        android:id="@+id/cet_auth_mobile"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <cn.yintech.cdam.view.ClearableEditText
        android:id="@+id/cet_auth_password"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="@dimen/spacing_row_vertical"/>

    <android.support.constraint.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="@dimen/spacing_24">

        <TextView
            android:id="@+id/tv_register"
            style="@style/ContentText.Large"
            android:text="@string/auth_register"
            android:textColor="@color/white"
            />

        <TextView
            android:id="@+id/tv_forget_password"
            style="@style/ContentText.Large"
            android:text="@string/auth_forget_pwd"
            android:textColor="@color/white"
            app:layout_constraintRight_toRightOf="parent"
            />
    </android.support.constraint.ConstraintLayout>

    <Button
        android:id="@+id/btn_login"
        style="@style/Button.Primary"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="@dimen/spacing_section_vertical"
        android:text="@string/auth_login"
        />
</LinearLayout>
```



# 分析

这应该是两个ClearableEditText中AutoCompleteTextView的id值相同的导致的。系统会在View的onSaveInstanceState中缓存View的状态，根据id来辨别View，因此相同id的View只保存最后一个的状态，之前的会被覆盖，所以最后都显示为同一个值。

# 解决

可为每个ClearableEditText中的AutoCompleteTextView动态分配一个id，做法如下
1. 在ClearableEditText的属性文件中加入id属性，如
```xml
<resources xmlns:tools="http://schemas.android.com/tools">
    <declare-styleable name="ClearableEditText">
        <!--省略其他-->
        <attr name="inputId" format="reference" />
    </declare-styleable>
</resources>
```

2. 在ClearableEditText中获取并添加id
```kotlin
private fun initView(context: Context, attrs: AttributeSet?, defStyleAttr: Int, defStyleRes: Int) {

        // init view
        val view = LayoutInflater.from(context).inflate(R.layout.view_clearable_edit_text, this, true)
        input = view.findViewById<AutoCompleteTextView>(R.id.et_clearable)
        clearImage = view.findViewById(R.id.iv_clearable)

        // init attr
        val ta = context.theme.obtainStyledAttributes(attrs, R.styleable.ClearableEditText, defStyleAttr, defStyleRes)
        try {
            val inputId = ta.getResourceId(R.styleable.ClearableEditText_inputId, 0x50)
            input.id = inputId
        } finally {
            ta.recycle()
        }
    }
```

3. 在res/value/ids.xml(没有这个文件的话，创建一个)中，加入各种类型的ClearableEditText需要的id.注意这里的值如0x51、0x52等其实可以不需要，因为应用为每个资源自动生成一个资源id，上面的代码中获取的就是资源id
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!--自定义ViewGroup中包含EditText，如果EditText相同则只缓存一个-->
    <item name="clearable_input_mobile" type="id">0x51</item>
    <item name="clearable_input_password" type="id">0x52</item>
    <item name="clearable_input_password_confirmed" type="id">0x53</item>
    <item name="clearable_input_password_new" type="id">0x54</item>
</resources>
```
4. 在布局中引用资源id 
```xml
    <!--省略-->
   <cn.yintech.cdam.view.ClearableEditText
        android:id="@+id/cet_auth_mobile"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:inputId="@id/clearable_input_mobile"/>

    <cn.yintech.cdam.view.ClearableEditText
        android:id="@+id/cet_auth_password"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="@dimen/spacing_row_vertical"
        app:inputId="@id/clearable_input_password"/>
    <!--省略-->
```