---
layout: post
title:  "JMeter的function扩展——原理解析"
subtitle: ""
keyword: "jmeter,function,principle,原理"
date: 2019-07-06
categories: jmeter
tags: jmeter,function,原理,custom function
background: ""
---

# 1. 介绍

在上一篇中，对JMeter的自定义function功能的使用方法做了介绍。但是众所周知JMeter压测模式是一个典型的多线程并发场景，function代码如何注入，如何实现线程安全等都要从源码中获得答案。

下面对JMeter的function实现原理进行分析

注：源码版本为3.3

# 2. 代码解析

首先看一下Function接口的接口说明，因为AbstractFunction抽象类的注释更清晰，先看下AbstractFunction的源码

```
public abstract class AbstractFunction implements Function {
    /**
     * <p><b>
     * N.B. execute() should be synchronized if function is operating with non-thread-safe
     * objects (e.g. operates with files).
     * </b></p>
     * JMeter ensures setParameters() happens-before execute(): setParameters is executed in main thread,
     * and worker threads are started after that.
     * @see Function#execute(SampleResult, Sampler)
     */
    @Override
    abstract public String execute(SampleResult previousResult, Sampler currentSampler) throws InvalidVariableException;
	
	.....
```

可以看出，在关键的setParameters、execute接口上，JMeter引擎使setParameters在主线程执行，并保证在execute方法前调用。而execute需要开发者自行保证线程安全，必定运行在多线程环境。

下面先解析一下setParameters如何工作：

## 2.1 setParameters方法

先观察入口StandardJMeterEngine.run方法：

```
    @Override
    public void run() {
        log.info("Running the test!");
        running = true;
		
		......
		
        JMeterContextService.startTest();
        try {
            PreCompiler compiler = new PreCompiler();
            test.traverse(compiler);
        } catch (RuntimeException e) {
		
		......
	}
```

在启动JMeter测试用例时，会使用预处理器(PreCompiler)先对JMeter脚本进行预处理

PreCompiler使用合适的HashTreeTraverser递归处理JMeter脚本（JMeter中称为HashTree），每个节点处理如下：

File: PreCompiler
```
@Override
    public void addNode(Object node, HashTree subTree) {
        if(isClientSide) {
			// Server-Client模式下的Client端处理
			......
        } else {
			// 单机模式
            if(node instanceof TestElement) {
                try {
                    replacer.replaceValues((TestElement) node);
                } catch (InvalidVariableException e) {
                    log.error("invalid variables in node {}", ((TestElement)node).getName(), e);
                }
            }
			......
        }
    }
```

上述代码看出，在单机模式与Server-Client模式下，处理方式略有不同，会屏蔽掉一些类型节点。这里我们只关注单机模式下的变量替换逻辑

replacer(ValueReplacer)的replaceValues对${...}变量进行初步处理：

File: ValueReplacer
```
	private Collection<JMeterProperty> replaceValues(PropertyIterator iter, ValueTransformer transform) throws InvalidVariableException {
        List<JMeterProperty> props = new LinkedList<>();
        while (iter.hasNext()) {
            JMeterProperty val = iter.next();
			......
            if (val instanceof StringProperty) {
                // Must not convert TestElement.gui_class etc
                if (!val.getName().equals(TestElement.GUI_CLASS) &&
                        !val.getName().equals(TestElement.TEST_CLASS)) {
                    val = transform.transformValue(val);
                    log.debug("Replacement result: {}", val);
                }
            } else if (val instanceof NumberProperty) {
			.....
			}
		}
		......
	}
```

现在来看一下transform变量（ReplaceStringWithFunctions类）的transformValue方法

File: ReplaceStringWithFunctions
```
    @Override
    public JMeterProperty transformValue(JMeterProperty prop) throws InvalidVariableException {
        JMeterProperty newValue = prop;
        getMasterFunction().clear();
        getMasterFunction().setParameters(prop.getStringValue());
        if (getMasterFunction().hasFunction()) {
            newValue = new FunctionProperty(prop.getName(), getMasterFunction().getFunction());
        }
        return newValue;
    }
```

现在到了一个关键的地方，getMasterFunction()实际返回CompoundVariable类实例，使用setParameters方法对表达式解析处理

File: CompoundVariable
```
    public void setParameters(String parameters) throws InvalidVariableException {
        this.rawParameters = parameters;
        if (parameters == null || parameters.length() == 0) {
            return;
        }

        compiledComponents = functionParser.compileString(parameters);
        if (compiledComponents.size() > 1 || !(compiledComponents.get(0) instanceof String)) {
            hasFunction = true;
        }
        permanentResults = null; // To be calculated and cached on first execution
        isDynamic = false;
        for (Object item : compiledComponents) {
            if (item instanceof Function || item instanceof SimpleVariable) {
                isDynamic = true;
                break;
            }
        }
    }
```

该处可以看出：

- FunctionParser使用compileString方法把字符串表达式再次解析，返回LinkedList<Object>类型数据存储表达式内的各型元素
- 设置hasFunction与isDynamic参数，将会决定上下文迭代时的变量处理行为

现在看下compileString的逻辑：

File: FunctionParser
```
    LinkedList<Object> compileString(String value) throws InvalidVariableException {
		......
		try {
			while (reader.read(current) == 1) {
				if (current[0] == '\\') { // Handle escapes
					......
				} else if (current[0] == '{' && previous == '$') {// found "${"
                    buffer.deleteCharAt(buffer.length() - 1);
                    if (buffer.length() > 0) {// save leading text
                        result.add(buffer.toString());
                        buffer.setLength(0);
                    }
                    result.add(makeFunction(reader));
                    previous = ' ';
                }
			}
		} catch (IOException e) {......}
		return result;
	}
```

现在来看一下makeFunction方法

File: FunctionParser
```Java
    /**
     * Compile a string into a function or SimpleVariable.
	 ......
	 */
	Object makeFunction(StringReader reader) throws InvalidVariableException {
		......
		while (reader.read(current) == 1) {
			if (current[0] == '\\') {
				......
			} else if (current[0] == '(' && previous != ' ') {
				String funcName = buffer.toString();
				function = CompoundVariable.getNamedFunction(funcName);
				if (function instanceof Function) {
					((Function) function).setParameters(parseParams(reader));
					......
					return function;
				}
			} else if (.....)
			......
		}
		......
	}
```

终于来到这一步，CompoundVariable.getNamedFunction(funcName)将针对不同的functionName构建不同的Function

在下一步中为这个Function实例设置参数值`((Function) function).setParameters(parseParams(reader));`

而这里的setParameters即我们写的Function扩展的setParameters方法实现

另外可以注意到，这一系列方法栈全都在main函数上运行，印证了开头AbstractFunction的类注释里说的。

## 2.2 execute方法

execute的方法栈从JMeterThread开始，即实际发起请求的线程。通过请求发起相关逻辑（因为发起请求逻辑与各个Sampler类型有关，这里不详述，后面再起专门的主题来讲）

在Sampler处理时，在处理参数时，最终会调用到FunctionProperty的getStringValue方法

File: FunctionProperty
```
	public String getStringValue() {
		......
		if (iter > testIteration || cacheValue == null) {
			testIteration = iter;
			cacheValue = function.execute();
		}
		return cacheValue;
	}
```

这里的function是一个组合的CompoundVariable，并调用execute方法

File: CompoundVariable
```
    public String execute(SampleResult previousResult, Sampler currentSampler) {
		......
        StringBuilder results = new StringBuilder();
        for (Object item : compiledComponents) {
            if (item instanceof Function) {
                try {
                    results.append(((Function) item).execute(previousResult, currentSampler));
                } catch (InvalidVariableException e) {
                    // TODO should level be more than debug ?
                    log.debug("Invalid variable: {}", item, e);
                }
            } else if (item instanceof SimpleVariable) {
                results.append(((SimpleVariable) item).toString());
            } else {
                results.append(item);
            }
        }
        if (!isDynamic) {
            permanentResults = results.toString();
        }
        return results.toString();
	}
```

该处方法处理所有compiledComponents（这里大家应该还有印象，在上述PreCompiler中生成过），并通过StringBuilder拼接组合最终的值

这里的`((Function) item).execute(previousResult, currentSampler)`中的item即自定义的Function。调用的execute方法即我们的Function扩展的execute方法实现

execute的调用都起于JMeterThread，调用链路上未对execute方法做同步动作，所以开发人员自己保障线程安全

而JMeterThread的生成与运行会在setParameters后

# 3. 总结

本次代码解析通过setParameters与execute方法的调用分析，印证了AbstractFunction的注释。

- Function机制的setParameters方法一定会在execute前，运行于JMeter脚本预解析阶段（非常靠前），并且在主线程上运行，不存在线程安全问题

- Function机制的execute方法运行在多线程环境下，并且内部没有做线程安全处理，需要在开发扩展时特别注意竞态条件并正确处理
