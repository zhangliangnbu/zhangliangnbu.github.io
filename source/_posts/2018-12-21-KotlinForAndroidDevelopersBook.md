---
title: Android开发者的Kotlin入门指南
date: 2018-12-21 08:43:10
tags:
- android
- kotlin
categories:
- Android开发
---

# 前言

这是一个本为Android开发者写的Kotlin入门书籍，适合想入门kotlin或使用还不够熟练的Android开发者们，可以让他们少走很多弯路。我已经使用Kotlin开发Android已有一年多了，快读读完后依旧有所收获。

- 英文原版：[官网书籍地址](https://leanpub.com/kotlin-for-android-developers)、[豆瓣地址](https://book.douban.com/subject/26916501/)
- 中文版：貌似还没有正式出版的中文版，只找到[Gitbook版本-《Kotlin for android developers》中文版翻译](https://wangjiegulu.gitbooks.io/kotlin-for-android-developers-zh/content/)

---

# val vs var

一般开发者当然知道不可变对象使用`val`，可变对象使用`var`。对二者的解释，书中有段话说得很好：

> 一个不可变对象意味着它在实例化之后就不能再去改变它的状态了。如果你需要一 个这个对象修改之后的版本，那就会再创建一个新的对象。这个让编程更加具有健壮性和预估性。在Java中，大部分的对象是可变的，那就意味着任何可以访问它这个对象的代码都可以去修改它，从而影响整个程序的其它地方。 
>
> 不可变对象也可以说是线程安全的，因为它们无法去改变，也不需要去定义访问控制，因为所有线程访问到的对象都是同一个。 
>
> 所以在Kotlin中，如果我们想使用不可变性，我们编码时思考的方式需要有一些改变。一个重要的概念是：尽可能地使用 val 。除了个别情况（ 特别是在Android中，有很多类我们是不会去直接调用构造函数的） ，大多数时候是可以的。 

---

# 网络请求

简单的网络请求：`val data = URL(url).readText()`。使用了Kotlin扩展函数，之前都没有使用过，汗。

---

# Anko库

简洁的线程切换。还是要早点使用Anko库啊，汗。

```kotlin
async() {
	Request(url).run()
	uiThread { longToast("Request performed") }
}
```



----

# 委托和泛型

委托和泛型估计是全书的难点和重点了。书中有个简单的例子，使用属性委托来封装SharePreference数据的获取与设置，很精彩！

属性委托的实现

```kotlin
class Preference<T>(val context: Context, val name:String, private val default: T)
    :ReadWriteProperty<Any?, T> {

    private val prefs by lazy {
        context.getSharedPreferences("default", Context.MODE_PRIVATE)
    }

    override fun getValue(thisRef: Any?, property: KProperty<*>): T {
        return findPreference(name, default)
    }

    override fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
        putPreference(name, value)
    }

    private fun<T> findPreference(name: String, default: T):T = with(prefs) {
        val res: Any = when(default) {
            is Int -> getInt(name, default)
            is Long -> getLong(name, default)
            is Float -> getFloat(name, default)
            is String -> getString(name, default)
            is Boolean -> getBoolean(name, default)
            else -> throw IllegalAccessException("this type can not be gotten from preference")
        }
        return@with res as T
    }

    private fun<U> putPreference(name: String, value: U) = with(prefs.edit()) {
        when(value) {
            is Int -> putInt(name, value)
            is Long -> putLong(name, value)
            is Float -> putFloat(name, value)
            is String -> putString(name, value)
            is Boolean -> putBoolean(name, value)
            else -> throw IllegalAccessException("this type can not be saved into preference")
        }.apply()
    }
}
```

封装为方法

```kotlin
object DelegateExt {
    fun<T : Any> preference(context: Context, name: String, default: T) =
        Preference(context, name, default)
}
```

使用

```kotlin
class KotlinForAndroidBookActivity : AppCompatActivity() {
    private var pName:String by DelegateExt.preference(this, "person_name", "default")
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_kotlin_for_android_book)

        // 设置
        pName = "liang"
        // 获取
        println(pName)
        // 抓取本地文件shared_prefs文件可以得到
        // <?xml version='1.0' encoding='utf-8' standalone='yes' ?>
        //<map>
        //    <string name="person_name">liang</string>
        //</map>
    }
}
```

相关源码请见我的GitHub项目[Android实践示例](https://github.com/zhangliangnbu/android-practice-demo)

---

# 参考

1. [Anko](https://github.com/Kotlin/anko)

---





