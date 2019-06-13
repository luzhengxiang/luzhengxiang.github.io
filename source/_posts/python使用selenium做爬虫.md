---
title: python学习selenium笔记
date: 2019-06-13 22:19:40
tags: [python, selenium]
---

# 安装
```
pip install selenium
```

同时还需要在你的电脑上安装浏览器驱动。安装你本机浏览器对应版本的浏览器驱动。<br>
Chromedriver: http://npm.taobao.org/mirrors/chromedriver/ <br>



# 具体操作
## 控制浏览器操作的一些方法
```
set_window_size()	设置浏览器的大小
back()	控制浏览器后退
forward()	控制浏览器前进
refresh()	刷新当前页面
clear()	清除文本
send_keys (value)	模拟按键输入
click()	单击元素
submit()	用于提交表单
get_attribute(name)	获取元素属性值
is_displayed()	设置该元素是否用户可见
size	返回元素的尺寸
text	获取元素的文本
```
## 鼠标事件
```
ActionChains(driver)	构造ActionChains对象
context_click()	执行鼠标悬停操作
move_to_element(above)	右击
double_click()	双击
drag_and_drop()	拖动
move_to_element(above)	执行鼠标悬停操作
context_click()	用于模拟鼠标右键操作， 在调用时需要指定元素定位
perform()	执行所有 ActionChains 中存储的行为，可以理解成是对整个操作的提交动作
```
## 键盘事件
```
常用键盘操作
send_keys(Keys.BACK_SPACE)	删除键（BackSpace）
send_keys(Keys.SPACE)	空格键(Space)
send_keys(Keys.TAB)	制表键(Tab)
send_keys(Keys.ESCAPE)	回退键（Esc）
send_keys(Keys.ENTER)	回车键（Enter）
组合键盘操作
send_keys(Keys.CONTROL,‘a’)	全选（Ctrl+A）
send_keys(Keys.CONTROL,‘c’)	复制（Ctrl+C）
send_keys(Keys.CONTROL,‘x’)	剪切（Ctrl+X）
send_keys(Keys.CONTROL,‘v’)	粘贴（Ctrl+V）
send_keys(Keys.F1…Fn)	键盘 F1…Fn

```

## 定位一组元素
```

```

## 多窗口切换
在页面操作过程中有时候点击某个链接会弹出新的窗口，这时就需要主机切换到新打开的窗口上进行操作。WebDriver提供了switch_to.window()方法，可以实现在不同的窗口之间切换。
```
current_window_handle	获得当前窗口句柄
window_handles	返回所有窗口的句柄到当前会话
switch_to.window()	用于切换到相应的窗口，与上一节的switch_to.frame()类似，前者用于不同窗口的切换，后者用于不同表单之间的切换。

```

## 调用JavaScript代码
虽然WebDriver提供了操作浏览器的前进和后退方法，但对于浏览器滚动条并没有提供相应的操作方法。在这种情况下，就可以借助JavaScript来控制浏览器的滚动条。WebDriver提供了execute_script()方法来执行JavaScript代码。
``` python 
直接翻到当前页面的页尾
browser.execute_script('window.scrollTo(0, document.body.scrollHeight)')
```

## cookie操作
```
get_cookies()	获得所有cookie信息
get_cookie(name)	返回字典的key为“name”的cookie信息
add_cookie(cookie_dict)	添加cookie。“cookie_dict”指字典对象，必须有name 和value 值
delete_cookie(name,optionsString)	删除cookie信息。“name”是要删除的cookie的名称，“optionsString”是该cookie的选项，目前支持的选项包括“路径”，“域”
delete_all_cookies()	删除所有cookie信息

```

## 关闭浏览器
```
close()	关闭单个窗口
quit()	关闭所有窗口

```

# 可能遇到的坑
因为sulenium是代码控制浏览器操作，代码执行很快，浏览器执行很慢，经常发生代码执行到这里，但是页面资源没有加载完的情况。所以需要 time.sleep(5) 让程序等待几秒钟。

参考链接：<br>
https://blog.csdn.net/weixin_36279318/article/details/79475388 