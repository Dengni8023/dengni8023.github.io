---
layout:     post
title:      iOS自定义安全键盘JGSourceBase解读
date:       2021-12-04
author:     dengni8023
catalog:    true
tags:
    - iOS
    - 自定义
    - 安全
    - 键盘
---

# JGSourceBase自定义安全键盘设计

iOS自定义安全键盘需要修改输入框的inputView属性，该属性一般在文本框初始化时指定。

	JGSSecurityKeyboar内布局包含
		- 顶部工具：JGSKeyboardToolbar
		- 键盘输入区域：JGSLetterKeyboard/JGSNumberKeyboard/JGSSymbolKeyboard

## 键盘设计思路

1. 根据设备（iPhone/iPad）屏幕的宽、键盘输入按钮的宽高比、按键水平以及垂直方向的间隔，分别计算出设备横竖屏状态的键盘高度。
2. 根据设备的safeArea、设备宽、键盘高，计算并缓存键盘输入区域的大小。
3. 键盘输入区域根据键盘显示区域大小、按键行、列展示情况计算每个按钮的布局。
4. 每个按钮JGSKeyboardKey继承UILabel实现点击、长按、双击操作处理。
5. 应用旋转时更新键盘高度，对应键盘输入区域变化，按键重新布局。
6. 键盘整体为自动布局，处理SafeArea边距等问题；键盘输入区域使用非自动布局，仅需要处理输入区域内部的布局计算处理。

## 键盘转屏适配

### 键盘高度修改

应用转屏时键盘布局适配关键在于键盘高度修改，键盘宽度系统自动根据屏幕宽度重新布局（键盘内部元素布局需开发处理）

1. 自定义键盘初始化时必须指定大于0的高度，否则后续键盘不会展示，更新高度也不会生效。
2. 键盘转屏需要监听系统通知进行键盘高度更新，可监听的通知包括：
	
		1、UIApplicationDidChangeStatusBarOrientationNotification
		2、UIKeyboardWillChangeFrameNotification
		3、UIKeyboardDidChangeFrameNotification
		4、UIKeyboardWillShowNotification

应用方向变化等导致键盘大小变化处理，需要考虑通知接收频率、及性能问题：

> 仅监听 UIKeyboardWillChangeFrameNotification 即可实现修改键盘高度操作
> 
> 通知执行顺序大概（键盘高度不实际更新是存在差异情况）如下：
> 
>  1、UIApplicationDidChangeStatusBarOrientationNotification：应用转屏后执行一次，最先执行
> 
> 2、UIKeyboardWillChangeFrameNotification：键盘弹出、应用转屏均会执行，如果收到通知不进行键盘高度更新，则仅执行一次，每更新一次则会重复执行一次
> 
> 3、UIKeyboardDidChangeFrameNotification：与keyboardWillChangeFrame配对执行，如果收到通知不进行键盘高度更新，则仅执行一次，每更新一次则会重复执行两次
> 
> 4、UIKeyboardWillShowNotification：键盘弹出、应用转屏均会执行，如果收到通知不进行键盘高度更新，则仅执行一次，每更新一次则会重复执行一次
	    
经测试：

> 1、在键盘高度不实际更新（调用更新方法，但是键盘实际高度不变）的情况下 UIKeyboardWillShowNotification 执行顺序在 UIKeyboardDidChangeFrameNotification 之后
> 
> 2、其他情况 UIKeyboardWillShowNotification、UIKeyboardDidChangeFrameNotification 执行顺序和高度更新的时机存在关联，可自行测试
	    
 收到 UIKeyboardWillChangeFrameNotification、UIKeyboardWillShowNotification、UIKeyboardDidChangeFrameNotification 时：
 
> 1、需要判断当前输入框是否有焦点，多个输入框同时存在时，系统通知可能多次发送
> 
> 2、每个通知处理方法均可执行键盘高度更新，UIKeyboardDidChangeFrameNotification 更新则会重复更新键盘高度，不建议在此处更新
> 
> 3、UIKeyboardWillShowNotification 通知中更新高度，则需要待转屏动画执行结束后键盘高度更新才会执行
> 
> <font color="red">综上，键盘高度更新在 UIKeyboardWillChangeFrameNotification 中进行最合适</font>
	    

UIKeyboardWillShowNotification中更新高度注意：

<font color="red">

转屏时 UIKeyboardWillChangeFrameNotification 通知首次执行在 UIKeyboardDidChangeFrameNotification 通知之后

如仅在UIKeyboardWillShowNotification更新键盘高度，键盘转屏动画完成之后才会执行键盘高度更新操作，键盘高度变化不流畅

</font>

### 键盘输入区域键盘布局刷新

由于键盘整体自动布局，键盘高度变化后，键盘输入区域frame会发生变化，根据键盘输入区域的整体大小、普通按钮的宽高比、功能按钮（输入法切换、Shift、Delete、Space、Return按钮）宽度与普通按钮按照一定比例处理，计算各按钮的frame。

1. 初始化时大概计算一次个按钮的布局。
2. 在键盘展示时、转屏等情况，在键盘的layoutSubviews方法中重新计算、更新键盘布局。

## JGSourceBase自定义安全键盘类说明

1. JGSBaseKeyboard：键盘输入区域键盘的父类
2. JGSKeyboardConstants：键盘布局、样式定义常、变量
3. JGSKeyboardToolbar：键盘顶部工具条，在键盘有标题时显示
4. JGSLetterKeyboard：英文字母输入键盘
5. JGSNumberKeyboard：数字输入键盘，支持身份证号输入键盘
6. JGSSecurityKeyboard：安全键盘入口文件
7. JGSSymbolKeyboard：符号输入键盘

## 键盘效果图（iPhone XS）

<table>
<tr>
<td><img src="http://github-blog.dengni8023.com/iOS自定义安全键盘JGSourceBase解读-1.png"/></td>
<td><img src="http://github-blog.dengni8023.com/iOS自定义安全键盘JGSourceBase解读-2.png"/></td>
<td><img src="http://github-blog.dengni8023.com/iOS自定义安全键盘JGSourceBase解读-3.png"/></td>
</tr>
</table>

![](http://github-blog.dengni8023.com/iOS自定义安全键盘JGSourceBase解读-4.png)

# 参考资料

1. [JGSourceBase自定义安全键盘Demo及源码](https://github.com/dengni8023/JGSourceBase.git)
