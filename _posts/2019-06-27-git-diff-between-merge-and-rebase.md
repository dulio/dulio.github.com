---
layout: post
title:  "Git的merge与rebase实践"
subtitle: ""
keyword: "git,merge,rebase"
date: 2019-06-27
categories: git
tags: git,merge,rebase
background: ""
---

# 1. 背景

网上教程讲git的merge与rebase时，大多讲了两种合并方式的原理。而一般不太讲哪种场景应该使用什么方式

下面通过一些实践来验证与探索这两种方式的使用场景

Git版本2.11.0

# 2. Merge与Rebase差异比较

假设当前git中有两个分支master与test分支，分别提交C、D与E、F如图：

```
       E---F (test)
      /
 A---B---C---D (master)
```
### 2.1 merge

在master分支上执行`git merge test`，并进行冲突合并

#### 2.1.1 情况1. C、D、E、F之间互相冲突

合并次数1次，手动解决冲突1次

git log结果为：

`A---B---C---D---E---F---G`

#### 2.1.2 情况2. C、D、E、F之间不互相冲突

合并次数1次，手动解决冲突0次

git log结果为：

`A---B---C---D---E---F---G'`

### 2.2 rebase

在master分支上执行`git rebase test`，并进行冲突合并

#### 2.2.1 情况1. C、D、E、F之间互相冲突，直接使用F节点内容，忽略C、D内容（即git rebase --skip）

合并次数2次，手动解决冲突2次

git log结果为：

`A---B---E---F`

#### 2.2.2 情况2. C、D、E、F之间互相冲突，合并F与C、D节点内容

合并次数2次，手动解决冲突2次

git log结果为：

`A---B---E---F---FC---FCD`

#### 2.2.3 情况3. C、D、E、F之间不冲突，自动合并F与C、D节点

合并次数2次，手动解决冲突0次

git log结果为：

`A---B---E---F---C---D`

当此时其他人使用`git pull --rebase`更新代码时，将把这些节点重新rebase一遍

# 3. 总结

从上述结果可以看出：

 - merge在处理合并时，比较的是最终节点内容。不管是否冲突，必然生成一个额外节点（合并节点，即parents=2），并且提交记录包含所有message
 - rebase在处理合并时，把基点加进分叉节点后，并一一合并节点内容。冲突情况时，可以选择skip掉某些节点，使skip的节点信息完全被忽略，也可以手动合并，产生新节点；rebase不冲突时不需要新增新节点，版本树清晰

## 对比表格

------

| 类型  | 优点 | 缺点 |
| ---- | ---- | ---- |
| Merge | 冲突时合并相对容易 | 产生额外节点污染版本树 |
| Rebase | 版本树较自然 | 在处理多版本冲突时较复杂 |

------

## 实践建议

假如大家使用Git多分支开发模式的建议：

1. 对于子分支（feature、hotfix类等私有分支）合并入主要分支（release、develop、master等公共分支）时**建议使用merge**，保证在冲突时，能以最终版本为准，减少分支合并工作量与其他人更新时的合并工作量

2. 在子分支（feature、hotfix类等私有分支）开发时，合并进更小的孙分支时，**建议使用rebase**保证一个清晰的版本树
