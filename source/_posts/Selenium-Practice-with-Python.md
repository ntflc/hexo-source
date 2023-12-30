---
title: 基于 Python 的 Selenium 自动化实践心得
date: 2017-03-17 14:20:14
categories:
- Test
tags:
- Test
- Selenium
- WebDriver
- Python
---

由于工作需要，从去年年底开始就在琢磨 WEB 自动化，去网上一搜就搜到了 Selenium。在这几个月的实践中，对 Selenium 也有了逐步的了解和自己的认识。一些基础的东西这里就不再赘述了，网上一搜一把，建议自行搜素。由于我更熟悉 Python 语言，所以本文均是基于 Python 的。建议刚接触 Selenium 的人多看官方文档，Python 版的地址如下：[http://selenium-python.readthedocs.io/](http://selenium-python.readthedocs.io/)。

<!-- more -->

# Selenium 常用操作
## 启动和关闭 Selenium WebDriver

在 Python 中，启动 Selenium WebDriver（这里以 Chrome WebDriver 为例）的方法如下：

``` python
from selenium import webdriver

# 启动 WebDriver
browser = webdriver.Chrome()
# 打开网站
browser.get("https://www.google.com")
# 关闭 WebDriver
browser.quit()
```

其中，使用 Chrome WebDriver 需要安装 Chrome 浏览器，并下载 Chrome WebDriver，并将 Chrome WebDriver 文件所在路径加到系统 PATH 中。如果不方便添加 PATH，可以在 `webdriver.Chrome()` 中填入绝对路径，如：

``` python
browser = webdriver.Chrome("/Users/ntflc/webdriver/chromedriver")
```

## 多 Tab 页面的切换

有的时候，自动化测试需要在多个 Tab 页面中切换。为了解决页面切换问题，我采用的方法是：新建页面时，记录该 Tab 页面的 handle:

``` python
def open_new_tab_and_switch(browser):
    # 获取当前 Tab 页数
    handle_cnt = len(browser.window_handles)
    # 新建 Tab 页面
    browser.execute_script("window.open('about:blank', '_blank');")
    # 跳转到新建的 Tab 页面
    browser.switch_to.window(browser.window_handles[handle_cnt])
    # 获取当前 Tab 页面的 handle
    current_handle = browser.current_window_handle
    # 返回当前 Tab 页面的 handle
    return current_handle
```

每次针对某个 Tab 页面操作时，使用 `browser.switch_to.window(handle)` 跳回对应 Tab 页面。

## iFrame 的跳转

刚使用 Selenium 的时候，经常会遇到一个问题，明明这个元素可见，但就是定位不到。这种时候，可以看看该元素是否在 iFrame 之中，如果在，需要先跳转到这个 iFrame 中才行，如：

``` python
# 跳转到 XPath 为 xpath 的 iFrame 中
browser.switch_to.frame(browser.find_element_by_xpath(xpath))
```

同时需要注意的是，如果后续操作不在该 iFrame 中，记得要跳回原页面。

## 下拉框的选择

对于下拉框，有时可以粗暴地用 `send_keys(value)` 来选择，但很多时候这样并不行，具体方法如下：

``` python
from selenium.webdriver.support.ui import Select

# 将 XPath 为 xpath 的下拉框选择值为 value 的选项
Select(browser.find_element_by_xpath(xpath).select_by_value(value)
```

## 上传文件

对于部分需要涉及到上传文件的地方，比如上传头像，其实原理就是找到对应的 input 元素，通过 `send_keys()` 将文件路径传递，如：

``` python
browser.find_element_by_xpath(xpath).send_keys(path_of_file)
```

其中，`xpath` 为 input 元素的 XPath，`path_of_file` 为文件的路径。

## 等待

刚接触 Selenium 的人经常会遇到这样的困惑：为什么我手动一行一行执行的时候正常，写到一个文件中执行就报错？原因是，前一行执行完，但页面却没有加载好，导致执行后一行时，相关元素还没有加载出来。起初，大家可能会想到 `time.sleep()`，但这毕竟不是一个好方法。后来查阅文档发现，Selenium 已经提供了等待相关的函数。

我这里总结了我常用的几个：

``` python
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as ec

# 等待 sec 秒，直到页面标题为 title
try:
    WebDriverWait(browser, sec).until(
        ec.title_is(title)
    )
except Exception as e:
    print e

# 等待 sec 秒，直到 Xpath 为 xpath 的元素可见
try:
    WebDriverWait(browser, sec).until(
        ec.visibility_of_element_located((By.XPATH, xpath))
    )
except Exception as e:
    print e

# 等待 sec 秒，直到 Xpath 为 xpath 的元素不可见
try:
    WebDriverWait(browser, sec).until(
        ec.invisibility_of_element_located((By.XPATH, xpath))
    )
except Exception as e:
    print e

# 等待 sec 秒，直到 Xpath 为 xpath 的元素展现
try:
    WebDriverWait(browser, sec).until(
        ec.presence_of_element_located((By.XPATH, xpath))
    )
except Exception as e:
    print e

# 等待 sec 秒，直到 Xpath 为 xpath 的元素中的值为 text
try:
    WebDriverWait(browser, sec).until(
        ec.text_to_be_present_in_element_value((By.XPATH, xpath), text)
    )
except Exception as e:
    print e
```

## 其他常见操作

``` python
# 设置 WebDriver 窗口页面分辨率为 1280x720
browser.set_window_size("1280", "720")

# 获取当前页面 url
browser.current_url

# 获取当前 handle 的 index
browser.window_handles.index(handle)

# 清空 XPath 为 xpath 的元素
browser.find_element_by_xpath(xpath).clear()

# 将 val 输入到 XPath 为 xpath 的元素中
browser.find_element_by_xpath(xpath).send_keys(val)

# 点击 XPath 为 xpath 的元素
browser.find_element_by_xpath(xpath).click()

# 获取 XPath 为 xpath 的元素的参数 key 的值
browser.find_element_by_xpath(xpath).get_attribute(key)

# 获取 XPath 为 xpath 的元素的的值
browser.find_element_by_xpath(xpath).text

# 点击坐标 (x,y)
element = browser.find_element_by_xpath("//body")
action = webdriver.common.action_chains.ActionChains(browser)
action.move_to_element_with_offset(element, x, y)
action.click()
action.perform()

# 获取当前浏览器的 User Agent 值
browser.execute_script("return navigator.userAgent")
```

# 无界面使用 Selenium

Selenium WebDriver 大多要求有界面，如果通过 ssh 远程操作，或者在 Jenkins 上部署，当运行到 `browser = webdriver.Chrome()` 的时候，就会报错。解决方法有两类：使用无需界面的 WebDriver、模拟生成界面。

## 无需界面的 WebDriver：PhantomJS

PhantomJS 是一个脚本化的无界面 WebKit，需要下载对应的 WebDriver，启动方法为：

``` python
browser = webdriver.PhantomJS()
```

但是，由于个人测试的时候，发现使用 PhantomJS WebDriver 代替 Chrome WebDriver 时，部分代码执行出错，因此放弃了。

## 模拟生成界面

这个是我现在项目使用的方法，可以模拟出一个界面，适用于将项目部署到 Jenkins 上。具体方法为：

``` python
from pyvirtualdisplay import Display

# 生成模拟界面，分辨率为 1280x720
display = Display(visible=False, size=(1280, 720))
display.start()

# 关闭模拟界面
display.stop()
```
