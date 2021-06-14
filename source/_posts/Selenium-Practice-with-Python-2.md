---
title: 基于Python的Selenium自动化实践心得2
date: 2017-04-25 10:03:04
categories:
- 测试
tags:
- 测试
- Selenium
- WebDriver
- Python
---

之前一篇文章《[基于Python的Selenium自动化实践心得](http://ntflc.com/2017/03/17/Selenium-Practice-with-Python/)》讲了Selenium的常用操作和无界面使用Selenium，这篇文章则重点讲讲Selenium与unittest结合。

关于unittest，这里不做过多介绍，不了解的可以去Google一下。

<!-- more -->

# unittest下的setUp()、tearDown()

在Python的unittest官方文档里（[https://docs.python.org/2/library/unittest.html](https://docs.python.org/2/library/unittest.html)），有关于setUp()、tearDown()、setUpClass()、tearDownClass()的介绍。

简单来说，在执行unittest最开始，会先执行setUpClass()，最后会执行tearDownClass()，也就是各执行一次。而每个case前都会执行setUp()，之后都会执行tearDown()。

举个例子：

``` python
class MyTest(unittest.TestCase):
    @classmethod
    def setUpClass(cls):
        print "setUpClass"

    def setUp(self):
        print "setUp"

    def testcase_1(self):
        print "testcase_1"
        self.assertEquals(1, 1)

    def testcase_2(self):
        print "testcase_2"
        self.assertEquals(2, 2)

    def tearDown(self):
        print "tearDown"

    @classmethod
    def tearDownClass(cls):
        print "tearDownClass"
```

这个例子的输出结果是：

```
setUpClass
setUp
testcase_1
tearDown
setUp
testcase_2
tearDown
tearDownClass
```

其中setUpClass()和tearDownClass()必须用@classmethod修饰，且第一个参数为cls。

# 在setUpClass()和tearDownClass()中调用Selenium

对于每一组unittest，只需要启动一次Selenium。因此将启动命令放到setUpClass()中，将关闭命令放到tearDownClass()中。即：

``` python
class MyTest(unittest.TestCase):
    @classmethod
    def setUpClass(cls):
        cls.driver = webdriver.Chrome()

    def testcase(self):
        self.driver.get("https://www.baidu.com/")
        btn_element = self.driver.find_element_by_id("su")
        btn_value = btn_element.get_attribute("value")
        self.assertEquals(btn_value, "百度一下")

    @classmethod
    def tearDownClass(cls):
        cls.driver.quit()
```

# 用例失败时自动截图

有的时候，用例会失败，这时我们希望能够自动获取截图。这里是针对每一个用例的，因此可以放在tearDown()中。截图的文件名可以用case的id，如上面的代码片段，文件名即为testcase。在case中，`self.id()`能获取到当前case的id，如上面的代码片段，testcase对应的id为`test.MyTest.testcase`。实际是我们只需要testcase，因此可以通过`testcase_name = self.id().split(".")[-1]`来获取。

代码示例：

``` python
class MyTest(unittest.TestCase):
    @classmethod
    def setUpClass(cls):
        cls.driver = webdriver.Chrome()

    def testcase(self):
        self.driver.get("https://www.baidu.com/")
        btn_element = self.driver.find_element_by_id("su")
        btn_value = btn_element.get_attribute("value")
        self.assertEquals(btn_value, "百度一下")

    def tearDown(self):
        if sys.exc_info()[0]:
            testcase_name = self.id().split(".")[-1]
            screenshot_path = "{}.jpg".format(testcase_name)
            self.driver.save_screenshot(screenshot_path)

    @classmethod
    def tearDownClass(cls):
        cls.driver.quit()
```

# unittest测试框架实现参数化

如果直接用unittest，同一个case想跑不同的数据，每一组数据都得对应一个独立的case函数。如我现在想要判断1+i(1<=i<=5)的结算结果是否正确，只能写成：

``` python
class MyTest(unittest.TestCase):
    def testcase_1(self):
        self.assertEquals(1 + 1, 2)

    def testcase_2(self):
        self.assertEquals(1 + 2, 3)

    def testcase_3(self):
        self.assertEquals(1 + 3, 4)

    def testcase_4(self):
        self.assertEquals(1 + 4, 5)

    def testcase_5(self):
        self.assertEquals(1 + 5, 6)
```

这显然是不可行的。

[parameterized](https://github.com/wolever/parameterized)是一个针对Python单元测试框架的参数化实现，除了支持unittest，也支持nose、py.test等其他框架。

对于unittest，针对上面那个例子，可以写成：

``` python
class MyTest(unittest.TestCase):
    @parameterized.expand([
        ("1add1", 1, 2),
        ("1add2", 2, 3),
        ("1add3", 3, 4),
        ("1add4", 4, 5),
        ("1add5", 5, 6)
    ])
    def testcase(self, name, i, rst):
        print self.id()
        self.assertEquals(1 + i, rst)
```

其中，@parameterized.expand中每个tulpe的第一个元素可以认为是case名。上述5个case的id会自动变成：

```
test.MyTest.testcase_0_1add1
test.MyTest.testcase_1_1add2
test.MyTest.testcase_2_1add3
test.MyTest.testcase_3_1add4
test.MyTest.testcase_4_1add5
```

有了parameterized，就可以针对一个case定制不同的数据了。
