---
title: SpannableString点击事件异常问题
date: 2018-11-04 11:42:17
tags:
- android
- SpannableString
categories:
- Android开发
---

# 问题描述

1. 注册页里需要用户同意两个协议《用户注册协议》和《隐私声明》，要求用户点击协议名称时分别进入对应的详细页面。如图所示。
    ![情景图](/images/spannablestring_click_event.jpg)
2. 点击图中区域1时，进入《用户注册协议》页面；点击区域2时，进入《隐私声明》页面；但点击区域3时却进入了《隐私声明》页面，这是不应该出现的，区域3是空白区域，不应该有点击效果。

<!-- more -->

# 代码实现

```kotlin
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        // ...省略其他代码
        // 构造SpannableString
        val userProtocol = getString(R.string.protocol_cdam_user)
        val privacyProtocol = getString(R.string.protocol_cdam_privacy)
        val protocols = getString(R.string.protocol_reception, userProtocol, privacyProtocol) + aidText
        val userProtocolStart = protocols.indexOf(userProtocol)
        val userProtocolEnd = userProtocolStart + userProtocol.length
        val privacyProtocolStart = protocols.indexOf(privacyProtocol)
        val privacyProtocolEnd = privacyProtocolStart + privacyProtocol.length
        val spsb = SpannableStringBuilder(protocols)
        // 设置“《用户注册协议》”点击事件
        spsb.setSpan(object : ProtocolClickableSpan() {
            override fun onClick(widget: View?) {
                WebHelper.toWebFragment((activity as BaseActivity), DisplayWebFragment(),
                        H5URL.USER_PROTOCOL, getString(R.string.auth_user_protocol), true)
            }
        }, userProtocolStart, userProtocolEnd, Spannable.SPAN_EXCLUSIVE_EXCLUSIVE)
        // 设置“《隐私声明》”点击事件
        spsb.setSpan(object : ProtocolClickableSpan() {
            override fun onClick(widget: View?) {
                WebHelper.toWebFragment((activity as BaseActivity), DisplayWebFragment(), H5URL.PRIVACY_PROTOCOL, isAddStack = true)
            }
        }, privacyProtocolStart, privacyProtocolEnd, Spannable.SPAN_EXCLUSIVE_EXCLUSIVE)
        tv_registration_protocols.movementMethod = LinkMovementMethod.getInstance()
        tv_registration_protocols.text = spsb

    }

    // ClickableSpan实现
    abstract inner class ProtocolClickableSpan: ClickableSpan() {
        override fun updateDrawState(ds: TextPaint?) {
            super.updateDrawState(ds)
            ds?.color = AppHelper.getColor(R.color.color_accent)
            ds?.textSize = activity!!.resources.getDimension(R.dimen.font_primary_normal)
            ds?.isUnderlineText = false
        }
    }
```
-- -- --

<br>

# 问题分析与解决

从代码来看，并不能找出什么问题。最近时间有限，没有深究SpannableString设置点击事件机制，猜测可能区域3延续了区域2的效果，因此可以在区域2后增加一个空格，并对空格附加空实现的点击事件，经过实践，可行。并最终姑且这样解决这个Bug。代码实现如下：

```kotlin
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
     
        // ...省略的代码

        val userProtocol = getString(R.string.protocol_cdam_user)
        val privacyProtocol = getString(R.string.protocol_cdam_privacy)
        // 增加一个空格
        val aidText = " " 
        val protocols = getString(R.string.protocol_reception, userProtocol, privacyProtocol) + aidText
        val userProtocolStart = protocols.indexOf(userProtocol)
        val userProtocolEnd = userProtocolStart + userProtocol.length
        val privacyProtocolStart = protocols.indexOf(privacyProtocol)
        val privacyProtocolEnd = privacyProtocolStart + privacyProtocol.length
        val aidTextStart = privacyProtocolEnd
        val aidTextEnd = protocols.length - 1
        val spsb = SpannableStringBuilder(protocols)
        spsb.setSpan(object : ProtocolClickableSpan() {
            override fun onClick(widget: View?) {
                WebHelper.toWebFragment((activity as BaseActivity), DisplayWebFragment(),
                        H5URL.USER_PROTOCOL, getString(R.string.auth_user_protocol), true)
            }
        }, userProtocolStart, userProtocolEnd, Spannable.SPAN_EXCLUSIVE_EXCLUSIVE)
        spsb.setSpan(object : ProtocolClickableSpan() {
            override fun onClick(widget: View?) {
                WebHelper.toWebFragment((activity as BaseActivity), DisplayWebFragment(), H5URL.PRIVACY_PROTOCOL, isAddStack = true)
            }
        }, privacyProtocolStart, privacyProtocolEnd, Spannable.SPAN_EXCLUSIVE_EXCLUSIVE)
        // 为空格增加空实现的点击事件，防止后面的空余部分延续之前的点击事件
        spsb.setSpan(object : ProtocolClickableSpan(){
            override fun onClick(widget: View?) {
            }
        },aidTextStart, aidTextEnd, Spannable.SPAN_INCLUSIVE_INCLUSIVE)
        tv_registration_protocols.movementMethod = LinkMovementMethod.getInstance()
        tv_registration_protocols.text = spsb
    }
```
-- -- --

<br>

# 深入研究-SpannableString设置ClickableSpan内部实现分析

先挖坑，以后有时间再填