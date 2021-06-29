---
title: Android序列化总结初步
date: 2019-02-18 14:57:23
tags:
- android
- Serializable
- Parcelable
categories:
- Android开发
---

# 前言

序列化概念请见[wikipedia](https://zh.wikipedia.org/wiki/%E5%BA%8F%E5%88%97%E5%8C%96)。

为什么需要序列化？

1. IPC需要。多进程对应的是不同的内存区域，不能通过共享内存通信，因此传递数据的对象必须可序列化。
2. 持久化到本地和网络传输。

---

# Serializable

1. 使用简单，开销大。
2. serialVersionUID保证同一个对象。
3. 对象序列化，大量I/O操作，临时变量多，频繁GC。

```kotlin
// 使用
data class SerializableUser(var userId: Int, var userName: String, var isMale: Boolean) : Serializable {
    companion object {
        private const val serialVersionUID = 1L
    }
}

// 序列化到disk
fun saveSerializable(so: Serializable) {
    val out = ObjectOutputStream(FileOutputStream(getFilePath(USER_CACHE_FILE)))
    out.writeObject(so)
    out.close()
}

// 反序列化
fun getSerializable(): Serializable {
    val `in` = ObjectInputStream(FileInputStream(getFilePath(USER_CACHE_FILE)))
    val so = `in`.readObject() as Serializable
    `in`.close()
    return so
}
```

---

# Parcelable

1. 使用略复杂，开销小。
2. 官方建议最好不要持久化到disk或网络传输，因为parceble数据一旦更改，反序列化就会失败，不能保持同一性。
3. 将一个完整的对象进行分解。

Parcel与Parcelable的关系？// todo

```kotlin
data class ParcelableBook(var number: String?, var name: String?, var price: Float): Parcelable {
    constructor(parcel: Parcel) : this(
            parcel.readString(),
            parcel.readString(),
            parcel.readFloat())

    override fun writeToParcel(parcel: Parcel, flags: Int) {
        parcel.writeString(number)
        parcel.writeString(name)
        parcel.writeFloat(price)
    }

    override fun describeContents(): Int {
        return 0
    }

    companion object CREATOR : Parcelable.Creator<ParcelableBook> {
        override fun createFromParcel(parcel: Parcel): ParcelableBook {
            return ParcelableBook(parcel)
        }

        override fun newArray(size: Int): Array<ParcelableBook?> {
            return arrayOfNulls(size)
        }
    }
}

/**
 * 持久化Parcelable对象到disk, 但官方不建议这样做，理由如下：
 * changes in the underlying implementation of any of the data in the Parcel can render older data unreadable.
 */
fun saveParcelable(parcelable: Parcelable) {
    val parcel = Parcel.obtain()
    parcel.writeParcelable(parcelable, 0)
    val out = DataOutputStream(FileOutputStream(getFilePath(USER_CACHE_FILE2)))
    out.write(parcel.marshall())
    parcel.recycle()
}

// 反序列化
fun getParcelable(): Parcelable? {
    val ino = DataInputStream(FileInputStream(getFilePath(USER_CACHE_FILE2)))
    val data = ino.readBytes()
    val parcel = Parcel.obtain()
    parcel.unmarshall(data, 0, data.size)
    parcel.setDataPosition(0)
    val parcelable =  parcel.readParcelable<Parcelable>(Thread.currentThread().contextClassLoader)
    ino.close()
    parcel.recycle()
    return parcelable
}
```

---

# 总结

本地IPC使用Parcelable，持久化到disk和网络传输使用Serializable

---

# 参考

1. 《Android开发艺术探索》，任玉刚，电子工业出版社。
2. [Cache your data with parcelable and disklrucache](https://gist.github.com/VladSumtsov/c4af1f4b8fe5099ca809)
3. [Parcelable vs Serializable](http://www.developerphil.com/parcelable-vs-serializable/)

---

