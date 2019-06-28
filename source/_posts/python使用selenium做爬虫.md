---
title: python学习selenium笔记
date: 2019-06-13 22:19:40
tags: [python, selenium]
---

# 前言
Selenium是一个浏览器自动化操作框架。可以模拟用户操作。这样我们就可以用selenium做很多事情了，测试自动化，爬虫。这里我接触和学习selenium也是使用来作为爬虫。

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


定位一个元素|定位多个元素|含义
-|-|-|
find_element_by_id | find_elements_by_id | 通过元素id定位
find_element_by_name | find_elements_by_name|通过元素name定位
find_element_by_xpath	|find_elements_by_xpath	|通过xpath表达式定位
find_element_by_link_text	|find_elements_by_link_tex	|通过完整超链接定位
find_element_by_partial_link_text	|find_elements_by_partial_link_text	|通过部分链接定位
find_element_by_tag_name	|find_elements_by_tag_name	|通过标签定位
find_element_by_class_name	|find_elements_by_class_name	|通过类名进行定位
find_elements_by_css_selector	|find_elements_by_css_selector	|通过css选择器进行定位



示例代码
页面html截图
![](/img/selenium/shili1.jpg)

``` python

from selenium import webdriver
import time

browser = webdriver.Firefox()
browser.execute_script('window.scrollTo(0, document.body.scrollHeight)')
time.sleep(5)


liuyan_list = browser.find_elements_by_xpath('//*[@id="j-comment-section"]/div/div[3]/div/ul[3]/li')
#获取用户留言
liuyan_content = liuyan_list[0].find_element_by_class_name('shiyongxinde.oh')
print(liuyan_content.text)
liuyan_shijian = liuyan_list[0].find_element_by_class_name('info-time').find_element_by_tag_name('a').text
#获取留言时间
print(liuyan_shijian)
liuyan_id = liuyan_list[0].find_element_by_class_name('woZanTong').get_attribute('appraiseid')
#获取留言id
print(liuyan_id)
nick_name =  liuyan_list[0].find_element_by_class_name('reply_avatar_userName').text
#获取昵称
print(nick_name)

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
## 页面加载慢
因为sulenium是代码控制浏览器操作，代码执行很快，浏览器执行很慢，经常发生代码执行到这里，但是页面资源没有加载完的情况。所以需要 time.sleep(5) 让程序等待几秒钟。

## 莫名其妙的异常
我代码写的很开心，执行一下突然报错。。。就是那种明明上一次启动还正常，可能写了一行注释回来执行就报错了。。。让我不知所措，最后是restart大法好。重启就好了，我没有找到发生的具体原因，目前是出现就重启。。。
``` python
/Users/luzhengxiang/anaconda3/envs/my_spider/bin/python /Users/luzhengxiang/PycharmProjects/autoSign/spiders/selenium_test/gome/guomei_selenium.py
Traceback (most recent call last):
  File "/Users/luzhengxiang/PycharmProjects/autoSign/spiders/selenium_test/gome/guomei_selenium.py", line 10, in <module>
    browser = webdriver.Chrome();
  File "/Users/luzhengxiang/anaconda3/envs/my_spider/lib/python3.7/site-packages/selenium/webdriver/chrome/webdriver.py", line 73, in __init__
    self.service.start()
  File "/Users/luzhengxiang/anaconda3/envs/my_spider/lib/python3.7/site-packages/selenium/webdriver/common/service.py", line 104, in start
    raise WebDriverException("Can not connect to the Service %s" % self.path)
selenium.common.exceptions.WebDriverException: Message: Can not connect to the Service chromedriver

```
![](/img/selenium/error1.jpg)

## 不同浏览器内核可能支持的方法不同
我最开始基于firefox写的代码，后面直接替换成了chrome。很多原本正常的方法就报错了。猜测原因就是可能不同浏览器对这个selenium的支持不一样导致的。

参考链接：<br>
https://blog.csdn.net/weixin_36279318/article/details/79475388 