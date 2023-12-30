---
title: 基于 Python 的 Selenium 自动化实践心得 II
date: 2017-04-25 10:03:04
categories:
- Test
tags:
- Test
- Selenium
- WebDriver
- Python
---

之前一篇文章 [基于Python的Selenium自动化实践心得](/2017/03/17/Selenium-Practice-with-Python/) 讲了 Selenium 的常用操作和无界面使用 Selenium，这篇文章则重点讲讲 Selenium 与 unittest 结合。

关于 unittest，这里不做过多介绍，不了解的可以去 Google 一下。

<!-- more -->

# unittest 下的 setUp() 和 tearDown()

在 Python 的 unittest 官方文档里（[https://docs.python.org/2/library/unittest.html](https://docs.python.org/2/library/unittest.html)），有关于 setUp()、tearDown()、setUpClass()、tearDownClass() 的介绍。

简单来说，在执行 unittest 最开始，会先执行 setUpClass()，最后会执行 tearDownClass()，也就是各执行一次。而每个 case 前都会执行 setUp()，之后都会执行 tearDown()。

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

其中 setUpClass() 和 tearDownClass() 必须用 `@classmethod` 修饰，且第一个参数为 `cls`。

# 在 setUpClass() 和 tearDownClass() 中调用 Selenium

对于每一组 unittest，只需要启动一次 Selenium。因此将启动命令放到 setUpClass() 中，将关闭命令放到 tearDownClass() 中。即：

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

有的时候，用例会失败，这时我们希望能够自动获取截图。这里是针对每一个用例的，因此可以放在 tearDown() 中。截图的文件名可以用 case 的 id，如上面的代码片段，文件名即为 testcase。在 case 中，`self.id()` 能获取到当前 case 的 id，如上面的代码片段，testcase 对应的 id 为 `test.MyTest.testcase`。实际是我们只需要 testcase，因此可以通过 `testcase_name = self.id().split(".")[-1]` 来获取。

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

# unittest 测试框架实现参数化

如果直接用 unittest，同一个 case 想跑不同的数据，每一组数据都得对应一个独立的 case 函数。如我现在想要判断 `1 + i (1 <= i <= 5)` 的结算结果是否正确，只能写成：

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

[parameterized](https://github.com/wolever/parameterized) 是一个针对 Python 单元测试框架的参数化实现，除了支持 unittest，也支持 nose、py.test 等其他框架。

对于 unittest，针对上面那个例子，可以写成：

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

其中，`@parameterized.expand` 中每个 tulpe 的第一个元素可以认为是 case 名。上述 5 个 case 的 id 会自动变成：

```
test.MyTest.testcase_0_1add1
test.MyTest.testcase_1_1add2
test.MyTest.testcase_2_1add3
test.MyTest.testcase_3_1add4
test.MyTest.testcase_4_1add5
```

有了 parameterized，就可以针对一个 case 定制不同的数据了。
