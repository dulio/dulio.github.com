---
layout: post
title:  "Java8的运行时常量池与MetaSpace"
subtitle: ""
keyword: "java,java8,metaspace,permgen"
date:   2018-08-03
categories: java
tags:	java
background: ""
---

# 起因

近期在看《深入理解Java虚拟机：JVM高级特性与最佳实践》，在看到运行时常量池这一章，有一个例子演示运行时常量池溢出，代码如下：

```
/**
 * 原文Java7 VM Args: -XX:PermSize=10M -XX:MaxPermSize=10M
 * Java8 VM Args: -XX:MetaspaceSize=10M -XX:MaxMetaspaceSize=10M
 */
public class RuntimeConstantPoolOOM {
    public static void main(String[] args) {
        List<String> list = new ArrayList<String>();
        long i = 0L;
        while(true) {
            list.add(String.valueOf(i++).intern());
        }
    }
}
```

我使用Java8来运行这段程序，自然Java8把PermGen移除，取而代之是Metaspace空间，所以使用下面这个JVM参数来跑。

问题来了，程序跑的结果是过了很久才停止运行，感觉不太对。

# 原因

在改JVM参数的时候，考虑的是PermGen迁移到了MetaSpace，那么只需要把参数改成MetaSpace就ok了，然而事实确不是这样。

那么马上我就想到是不是运行时常量池并非从PermGen移到MetaSpace了？既然没有移到MetaSpace，那么很有可能移到Heap区了。

于是经过几次尝试，最后使用`-Xms10m -Xmx10m -XX:-UseGCOverheadLimit`重现了这段程序的目的，最后报错信息是`Exception in thread "main" java.lang.OutOfMemoryError: Java heap space`

移除PermGen原因，见原文[JEP 122: Remove the Permanent Generation](http://openjdk.java.net/jeps/122)

最终PermGen的方法区移至 Metaspace；字符串常量池移至 Java Heap。
