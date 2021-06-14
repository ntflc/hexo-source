---
title: 基于Python的Selenium自动化实践心得
date: 2017-03-17 14:20:14
categories:
- 测试
tags:
- 测试
- Selenium
- WebDriver
- Python
---

由于工作需要，从去年年底开始就在琢磨WEB自动化，去网上一搜就搜到了Selenium。在这几个月的实践中，对Selenium也有了逐步的了解和自己的认识。一些基础的东西这里就不再赘述了，网上一搜一把，建议自行搜素。由于我更熟悉Python语言，所以本文均是基于Python的。建议刚接触Selenium的人多看官方文档，Python版的地址如下：[http://selenium-python.readthedocs.io/](http://selenium-python.readthedocs.io/)。

<!-- more -->

# Selenium常用操作
## 启动和关闭Selenium WebDriver

在Python中，启动Selenium WebDriver（这里以Chrome WebDriver为例）的方法如下：

``` python
from selenium import webdriver

# 启动WebDriver
browser = webdriver.Chrome()
# 打开网站
browser.get("https://www.google.com")
# 关闭WebDriver
browser.quit()
```

其中，使用Chrome WebDriver需要安装Chrome浏览器，并下载Chrome WebDriver，并将Chrome WebDriver文件所在路径加到系统PATH中。如果不方便添加PATH，可以在`webdriver.Chrome()`中填入绝对路径，如：

``` python
browser = webdriver.Chrome("/Users/ntflc/webdriver/chromedriver")
```

## 多Tab页面的切换

有的时候，自动化测试需要在多个Tab页面中切换。为了解决页面切换问题，我采用的方法是：新建页面时，记录该Tab页面的handle:

``` python
def open_new_tab_and_switch(browser):
    # 获取当前Tab页数
    handle_cnt = len(browser.window_handles)
    # 新建Tab页面
    browser.execute_script("window.open('about:blank', '_blank');")
    # 跳转到新建的Tab页面
    browser.switch_to.window(browser.window_handles[handle_cnt])
    # 获取当前Tab页面的handle
    current_handle = browser.current_window_handle
    # 返回当前Tab页面的handle
    return current_handle
```

每次针对某个Tab页面操作时，使用`browser.switch_to.window(handle)`跳回对应Tab页面。

## iFrame的跳转

刚使用Selenium的时候，经常会遇到一个问题，明明这个元素可见，但就是定位不到。这种时候，可以看看该元素是否在iFrame之中，如果在，需要先跳转到这个iFrame中才行，如：

``` python
# 跳转到XPath为xpath的iFrame中
browser.switch_to.frame(browser.find_element_by_xpath(xpath))
```

同时需要注意的是，如果后续操作不在该iFrame中，记得要跳回原页面。

## 下拉框的选择

对于下拉框，有时可以粗暴地用`send_keys(value)`来选择，但很多时候这样并不行，具体方法如下：

``` python
from selenium.webdriver.support.ui import Select

# 将XPath为xpath的下拉框选择值为value的选项
Select(browser.find_element_by_xpath(xpath).select_by_value(value)
```

## 上传文件

对于部分需要涉及到上传文件的地方，比如上传头像，其实原理就是找到对应的input元素，通过`send_keys()`将文件路径传递，如：

``` python
browser.find_element_by_xpath(xpath).send_keys(path_of_file)
```

其中，`xpath`为input元素的XPath，`path_of_file`为文件的路径。

## 等待

刚接触Selenium的人经常会遇到这样的困惑：为什么我手动一行一行执行的时候正常，写到一个文件中执行就报错？原因是，前一行执行完，但页面却没有加载好，导致执行后一行时，相关元素还没有加载出来。起初，大家可能会想到`time.sleep()`，但这毕竟不是一个好方法。后来查阅文档发现，Selenium已经提供了等待相关的函数。

我这里总结了我常用的几个：

``` python
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as ec

# 等待sec秒，直到页面标题为title
try:
    WebDriverWait(browser, sec).until(
        ec.title_is(title)
    )
except Exception as e:
    print e

# 等待sec秒，直到Xpath为xpath的元素可见
try:
    WebDriverWait(browser, sec).until(
        ec.visibility_of_element_located((By.XPATH, xpath))
    )
except Exception as e:
    print e

# 等待sec秒，直到Xpath为xpath的元素不可见
try:
    WebDriverWait(browser, sec).until(
        ec.invisibility_of_element_located((By.XPATH, xpath))
    )
except Exception as e:
    print e

# 等待sec秒，直到Xpath为xpath的元素展现
try:
    WebDriverWait(browser, sec).until(
        ec.presence_of_element_located((By.XPATH, xpath))
    )
except Exception as e:
    print e

# 等待sec秒，直到Xpath为xpath的元素中的值为text
try:
    WebDriverWait(browser, sec).until(
        ec.text_to_be_present_in_element_value((By.XPATH, xpath), text)
    )
except Exception as e:
    print e
```

## 其他常见操作

``` python
# 设置WebDriver窗口页面分辨率为1280x720
browser.set_window_size("1280", "720")

# 获取当前页面url
browser.current_url

# 获取当前handle的index
browser.window_handles.index(handle)

# 清空XPath为xpath的元素
browser.find_element_by_xpath(xpath).clear()

# 将val输入到XPath为xpath的元素中
browser.find_element_by_xpath(xpath).send_keys(val)

# 点击XPath为xpath的元素
browser.find_element_by_xpath(xpath).click()

# 获取XPath为xpath的元素的参数key的值
browser.find_element_by_xpath(xpath).get_attribute(key)

# 获取XPath为xpath的元素的的值
browser.find_element_by_xpath(xpath).text

# 点击坐标(x,y)
element = browser.find_element_by_xpath("//body")
action = webdriver.common.action_chains.ActionChains(browser)
action.move_to_element_with_offset(element, x, y)
action.click()
action.perform()

# 获取当前浏览器的User Agent值
browser.execute_script("return navigator.userAgent")
```

# 无界面使用Selenium

Selenium WebDriver大多要求有界面，如果通过ssh远程操作，或者在Jenkins上部署，当运行到`browser = webdriver.Chrome()`的时候，就会报错。解决方法有两类：使用无需界面的WebDriver、模拟生成界面。

## 无需界面的WebDriver：PhantomJS

PhantomJS是一个脚本化的无界面WebKit，需要下载对应的WebDriver，启动方法为：

``` python
browser = webdriver.PhantomJS()
```

但是，由于个人测试的时候，发现使用PhantomJS WebDriver代替Chrome WebDriver时，部分代码执行出错，因此放弃了。

## 模拟生成界面

这个是我现在项目使用的方法，可以模拟出一个界面，适用于将项目部署到Jenkins上。具体方法为：

``` python
from pyvirtualdisplay import Display

# 生成模拟界面，分辨率为1280x720
display = Display(visible=False, size=(1280, 720))
display.start()

# 关闭模拟界面
display.stop()
```
