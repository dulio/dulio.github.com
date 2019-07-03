---
layout: post
title:  "JMeter的function扩展——介绍"
subtitle: ""
keyword: "jmeter,function"
date: 2019-07-03
categories: jmeter
tags: jmeter,function
background: ""
---

# 1. 介绍

近期需要在JMeter中集成一些公司业务相关的功能，如加密解密，API签名生成等相关功能。所以想到JMeter的Custom Function（下面简称function）功能来实现。

下面来简单介绍一下JMeter function

# 2. function扩展开发

开发一个JMeter的function非常容易，只需要满足三个条件：

- 扩展类，实现Function接口，或继承AbstractFunction

- 扩展类的命名空间需要包含.functions. （可以通过JMeter配置中classfinder.functions.contain项来修改）

- 把编译后的jar包放到JMeter目录lib/ext下，重启JMeter

想看下是否生效了？可以打开JMeter GUI，在Options->Function Helper Dialog选项查看所有自定义function

# 3. DES加密函数示例

下面来写一个示例，用于des加密，DesEncryption类自行编写：

### 3.1 扩展代码

```Java

public class DesEncryptPlugin extends AbstractFunction {
    Logger LOGGER = Logger.getLogger(DesEncryptPlugin.class);
    private static final String functionName = "__desEncrypt";
    private Object[] values;

    private final DesEncryption encryptionService = new DesEncryption(false);

    @Override
    public String execute(SampleResult sampleResult, Sampler sampler) throws InvalidVariableException {
        String content = ((CompoundVariable) values[0]).execute().trim();
        String desKey = "";

        String result;
        if (values.length >= 2){
            desKey = ((CompoundVariable) values[1]).execute();
            result = encryptionService.encrypt(content, encryptionService.getDesKey(desKey));
        } else {
            result = encryptionService.encrypt(content);
        }

        LOGGER.debug("execute DesEncryptPlugin: " + content + " " + desKey);
        return result;
    }

    @Override
    public void setParameters(Collection<CompoundVariable> collection) throws InvalidVariableException {
        checkParameterCount(collection, 1, 2);
        values = collection.toArray();
    }

    @Override
    public String getReferenceKey() {
        return functionName;
    }

    @Override
    public List<String> getArgumentDesc() {
        return new ArrayList<String>() {{
            add("Encrypt Content");
            add("DES Key(Optional)");
        }};
    }
}
```

### 3.2 代码要点

- 包管理时需要引入JMeter_core包，scope为provided，版本与最终运行版本一致

- 实现getReferenceKey方法：配置函数名

- 实现getArgumentDesc方法：设置函数参数的说明文字（非必要）

- 实现setParameters方法：从脚本中注册的表达式中获得参数

- 实现execute方法：获得返回值的主体方法，包含主要逻辑

### 3.3 使用方法

在JMeter中使用表达式直接调用

`${__desEncrypt(${content},testDesK)}`

- 注意不需要加引号

- 参数用逗号分隔，如果参数里也有逗号，需要使用\,转义

- 自定义function支持使用变量，具体实现原理下次再分析