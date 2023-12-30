---
title: TestNG 失败重跑（支持使用 dataProvider 的参数化用例）
date: 2018-10-18 20:36:41
categories:
- Test
tags:
- Test
- TestNG
---

最近在用 Java+TestNG+Maven 写 UI 自动化。因为之前用惯了 Python 的测试框架，失败重跑装个插件（[flaky](https://github.com/box/flaky) 或者 [pytest-rerunfailures](https://github.com/pytest-dev/pytest-rerunfailures)）就行。而 TestNG 的失败重跑需要自己重新方法，并且网上搜了很多资料，针对使用了 `dataProvider` 的参数化用例都存在一些问题。因此希望这篇文章能对需要的人起到帮助。

<!-- more -->

总体方案与网上能搜到大同小异：

1. 新建一个继承 `IRetryAnalyzer` 接口的类，这个类主要用于写失败重跑的规则
2. 新建一个继承 `IAnnotationTransformer` 接口的类，用于监听事件
3. 在 TestNG 的 XML 文件中配置监听

那么就一步一步来。

# 新建 `Retry` 类

首先，新建 `Retry` 类，继承 `IRetryAnalyzer` 接口，并自定义重跑规则：

``` java
package com.ntflc.listener;

import org.testng.IRetryAnalyzer;
import org.testng.ITestResult;

public class Retry implements IRetryAnalyzer {
    private int retryCnt = 0;
    private int maxRetryCnt = 2;

    @Override
    public boolean retry(ITestResult result) {
        if (retryCnt < maxRetryCnt) {
            retryCnt++;
            return true;
        }
        return false;
    }
}
```

其中 `maxRetryCnt` 是每个用例最多重试的次数（不包括第 1 次执行），`retryCnt` 是已经重跑的次数。`retry` 方法判断如果已经重跑的次数 `retryCnt` 小于设定的总次数，则返回 `true` 进行重跑，同时 `retryCnt` 加 1；否则返回 `false` 不再重跑。

下文均以最多重跑 2 次为例。

# 新建 `RetryListener` 类

然后，新建 `RetryListener` 类，继承 `IAnnotationTransformer` 接口：

``` java
package com.ntflc.listener;

import java.lang.reflect.Constructor;
import java.lang.reflect.Method;
import org.testng.IAnnotationTransformer;
import org.testng.IRetryAnalyzer;
import org.testng.annotations.ITestAnnotation;

public class RetryListener implements IAnnotationTransformer {
    public void transform(ITestAnnotation annotation, Class testClass, Constructor testConstructor, Method testMethod) {
        IRetryAnalyzer retry = annotation.getRetryAnalyzer();
        if (retry == null) {
            annotation.setRetryAnalyzer(Retry.class);
        }
    }
}
```

# 配置监听

最后，在 TestNG 的 XML 文件中配置监听：

``` xml
<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE suite SYSTEM "http://testng.org/testng-1.0.dtd" >
<suite name="MyTest">
    <listeners>
        <listener class-name="com.ntflc.listener.RetryListener"/>
    </listeners>
    <test name="Test1">
        ...
    </test>
</suite>
```

这样，对于非使用了 `dataProvider` 的用例，如果失败会进行重跑，最多跑 2 次。

# 问题与解决

## 存在的问题

上面的做法是网上绝大多数文章的全部内容，但对于一个使用了 `dataProvider` 的用例，因为这个用例是一个标记为 `@Test` 的方法，会共用 `Retry` 的 `retryCnt`，即整个方法的所有参数化用例，总共只会重跑 2 次。例如一个参数化用例有 3 组参数，如果全部正确，结果是：

```
Test1: success
Test2: success
Test3: success
```

如果第 1 个用例失败 1 次（第 2 次成功），第 2 个用例如果均失败，总共只跑了 2 次。因为第 1 个用例第 1 次失败时，`retryCnt` 为 0 并进行重跑；第 2 个用例第 1 次失败后，`retryCnt` 为 1 并进行重跑；第 2 个用例第 2 次失败后，`retryCnt` 为 2 因此不再重跑。即：
```
Test1: failed -> skipped
Test1: suceees
Test2: failed -> skipped
Test2: failed
Test3: failed
Test3: failed
```

至于为什么 `Test3` 也重跑了 1 次，这里不太清楚，因为 `Test3` 第 1 次失败时，`retryCnt` 为 2 返回的是 `false`，不应该再进行重跑。这里不清楚是不是 TestNG 的 Bug。

对此，网上有部分文章，会在 `Retry` 的 `return false;` 前设置 `retryCnt = 0;`，即：

``` java
package com.ntflc.listener;

import org.testng.IRetryAnalyzer;
import org.testng.ITestResult;

public class Retry implements IRetryAnalyzer {
    private int retryCnt = 0;
    private int maxRetryCnt = 2;

    @Override
    public boolean retry(ITestResult result) {
        if (retryCnt < maxRetryCnt) {
            retryCnt++;
            return true;
        }
        retryCnt = 0;
        return false;
    }
}
```

但这样只有在 `retryCnt` 达到 `maxRetryCnt` 后才会重置。即对于每个参数化用例都失败的情况，这样是没问题的：

```
Test1: failed -> skipped
Test1: failed -> skipped
Test1: failed
Test2: failed -> skipped
Test2: failed -> skipped
Test2: failed
Test3: failed -> skipped
Test3: failed -> skipped
Test3: failed
```

但如果一旦有一个参数化用例没有跑到 `maxRetryCnt` 的次数，`retryCnt` 就不会重置为 0，如：

```
Test1: failed -> skipped
Test1: success
Test2: failed -> skipped
Test2: failed
Test3: success
```

因为 `Test1` 失败了一次，重跑后 `retryCnt` 为 1。当 `Test2` 第 1 次失败时，此时 `retryCnt` 为 1（没有重置为 0），可以重跑，但返回 `true` 后 `retryCnt` 就变为 2 了，从而导致第 2 次失败时，判断为 `false` 不再重跑。因此 `Test2` 只重跑了 1 次就直接标为失败。

## 解决方法

解决上述问题的方法其实很简单，即每个参数化的用例结束（无论成功、失败）后，重置 `retryCnt`。

这里我们先在 `Retry` 中增加一个重置 `retryCnt` 的方法 `reset`：

``` java
package com.ntflc.listener;

import org.testng.IRetryAnalyzer;
import org.testng.ITestResult;

public class Retry implements IRetryAnalyzer {
    private int retryCnt = 0;
    private int maxRetryCnt = 2;

    @Override
    public boolean retry(ITestResult result) {
        if (retryCnt < maxRetryCnt) {
            retryCnt++;
            return true;
        }
        return false;
    }
    
    // 用于重置 retryCnt
    public void reset() {
        retryCnt = 0;
    }
}
```

然后新建 `TestngListener` 类，继承 `TestListenerAdapter` 类，并重写 `onTestSuccess` 和 `onTestFailure` 方法：

``` java
package com.ntflc.listener;

import org.testng.TestListenerAdapter;
import org.testng.ITestResult;

public class TestngListener extends TestListenerAdapter {
    @Override
    public void onTestSuccess(ITestResult tr) {
        super.onTestSuccess(tr);
        // 对于 dataProvider 的用例，每次成功后，重置 Retry 次数
        Retry retry = (Retry) tr.getMethod().getRetryAnalyzer();
        retry.reset();
    }

    @Override
    public void onTestFailure(ITestResult tr) {
        super.onTestFailure(tr);
        // 对于 dataProvider 的用例，每次失败后，重置 Retry 次数
        Retry retry = (Retry) tr.getMethod().getRetryAnalyzer();
        retry.reset();
    }
}
```

最后，在 TestNG 的 XML 中配置该监听：

``` xml
<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE suite SYSTEM "http://testng.org/testng-1.0.dtd" >
<suite name="MyTest">
    <listeners>
        <listener class-name="com.ntflc.listener.RetryListener"/>
        <listener class-name="com.ntflc.listener.TestngListener"/>
    </listeners>
    <test name="Test1">
        ...
    </test>
</suite>
```

这样，对于使用了 `dataProvider` 用例中的每一个参数化用例，都会最多跑 2 次，无论最后成功还是失败，都会重置 `Retry` 中的 `retryCnt` 以保证下一个参数化用例开始时，`retryCnt` 为初始状态。

以上就是本文的全部内容，由于本人使用 TestNG 时间较短，Java 基础也比较薄弱，难免会有疏漏，欢迎交流。
