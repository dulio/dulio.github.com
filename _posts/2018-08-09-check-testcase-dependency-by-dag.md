---
layout: post
title:  "基于有向图的测试用例依赖检测"
subtitle: ""
keyword: "DAG,testcase,test automation,testcase dependency check"
date:   2018-08-09
categories: test automation
tags:	dag,test automation
background: ""
---
# 背景

近期在实施一套测试自动化平台的项目，遇到一个工程问题：平台支持断言、用例参数的动态化，支持从历史用例、前置动作里获取结果，也支持实时运行依赖的用例。这样的功能带来一个问题，如果两个用例之间互相依赖，如A依赖B，B又反过来依赖A，将会导致这个测试集死循环。

# 思路与基础知识

这是一个典型的依赖检查，可以考虑用图来解决，对应的是有向无图（Directed Graph）。

图的定义：是由顶点的有穷非空集合和顶点之间边的集合组成，通常表示为：G(V, E)，其中，G表示一个图，V是图G中顶点的集合，E是图G中边的集合。按照边的有无方向性分为无向图和有向图。

在有向图中，无法从某个顶点出发经过若干条边回到该点，则这个图是一个有向无环图（DAG图），见下图。

[<img src="//assets/images/1533815166312.png">](DAG)

# 工具库与代码编写

Google Guava里提供了图相关的库可以使用，所以基于Guava进行开发。代码非常简洁，主要根据TestNG的不同suite，缓存了不同的图。

```
public final class TestCaseCycleChecker {
    private static ConcurrentHashMap<String, MutableGraph<Integer>> graphs = new ConcurrentHashMap<>();

    public static void addNode(String suite, Integer testCaseId) {
        getOrCreateGraph(suite).addNode(testCaseId);
    }

    public static void addEdge(String suite, Integer nodeU, Integer nodeV) {
        getOrCreateGraph(suite).putEdge(nodeU, nodeV);
    }

    public static boolean checkCycle(String suite) {
        return Graphs.hasCycle(getOrCreateGraph(suite));
    }

    private static MutableGraph<Integer> getOrCreateGraph(String suite) {
        MutableGraph<Integer> graph = graphs.getOrDefault(suite, null);
        if (null == graph) {
            graph = GraphBuilder
                    .directed()
                    .allowsSelfLoops(true)
                    .build();
            graphs.put(suite, graph);
        }
        return graph;
    }
}
```

无环测试：
```
String suite = "uncycled";
TestCaseCycleChecker.addNode(suite, 1);
TestCaseCycleChecker.addNode(suite, 2);
TestCaseCycleChecker.addNode(suite, 3);
TestCaseCycleChecker.addEdge(suite, 1, 2);
TestCaseCycleChecker.addEdge(suite, 2, 3);
TestCaseCycleChecker.addEdge(suite, 1, 3);
Assert.assertTrue(!TestCaseCycleChecker.checkCycle(suite));
```

有环测试：
```
String suite = "cycled";
TestCaseCycleChecker.addNode(suite, 1);
TestCaseCycleChecker.addNode(suite, 2);
TestCaseCycleChecker.addNode(suite, 3);
TestCaseCycleChecker.addEdge(suite, 1, 2);
TestCaseCycleChecker.addEdge(suite, 2, 3);
TestCaseCycleChecker.addEdge(suite, 3, 1);
Assert.assertTrue(TestCaseCycleChecker.checkCycle(suite));
```