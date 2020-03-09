---
layout: post
title:  "【软件工程】《设计模式》书摘"
crawlertitle: "《设计模式》"
summary: "设计模式——可复用面向对象软件的基础"
date:   2020-02-02 09:00:00 +0800
categories: posts
tags: '阅读'
author: xusc
---

可复用面向对象软件的基础

#### 设计模式思想
设计模式四个基本要素：模式名称、问题、解决方案、效果。

> MVC的主要关系由Observer、Composite、Strategy三个设计模式给出。

模式分类
1. 目的准则，即模式是用来完成什么工作的。
   - 创建型
   - 结构型
   - 行为型
2. 范围准则，即模式主要是用于类还是对象。
   - 类模式
   - 对象模式

设计模式怎样解决问题
1. 寻找合适的对象：设计模式帮你确定并不明显的抽象和描述这些抽象的对象。
2. 决定对象的粒度。
3. 指定对象接口：通过确定接口的主要组成成分及接口发送的数据类型，来帮助你定义接口。
4. 描述对象的实现。
5. 运用复用机制。
6. 关联运行时刻和编译时刻的结构。
7. 设计应支持变化。

设计模式所支持的设计的可变方面

<table border="1">
    <tr>
        <th>类型</th>
        <th>设计模式</th>
        <th>可变的方面</th>
    </tr>
    <tr>
        <td rowspan="5">创建型</td>
        <td>抽象工厂</td>
        <td align="right">产品对象家族</td>
    </tr>
    <tr>
        <td>建造者</td>
        <td align="right">如何创建一个组合对象</td>
    </tr>
    <tr>
        <td>工厂模式</td>
        <td align="right">被实例化的子类</td>
    </tr>
    <tr>
        <td>原型</td>
        <td align="right">被实例化的类</td>
    </tr>
    <tr>
        <td>单例</td>
        <td align="right">一个类的唯一实例</td>
    </tr>
    <tr>
        <td rowspan="7">结构型</td>
        <td>适配器</td>
        <td align="right">对象的接口</td>
    </tr>
    <tr>
        <td>桥接</td>
        <td align="right">对象的实现</td>
    </tr>
    <tr>
        <td>组合</td>
        <td align="right">一个对象的结构和组成</td>
    </tr>
    <tr>
        <td>装饰者</td>
        <td align="right">对象的生成，不生成子类</td>
    </tr>
    <tr>
        <td>外观</td>
        <td align="right">一个子系统的接口</td>
    </tr>
    <tr>
        <td>享元</td>
        <td align="right">对象的存储开销</td>
    </tr>
    <tr>
        <td>代理</td>
        <td align="right">如何访问一个对象；该对象的位置</td>
    </tr>
    <tr>
        <td rowspan="11">行为型</td>
        <td>职责链</td>
        <td align="right">满足一个请求的对象</td>
    </tr>
    <tr>
        <td>命令</td>
        <td align="right">何时、怎样满足一个请求</td>
    </tr>
    <tr>
        <td>解释器</td>
        <td align="right">一个语言的文法及解释</td>
    </tr>
    <tr>
        <td>迭代器</td>
        <td align="right">如何遍历、访问一个聚合的各元素</td>
    </tr>
    <tr>
        <td>中介者</td>
        <td align="right">对象怎样交互、和谁交互</td>
    </tr>
    <tr>
        <td>备忘录</td>
        <td align="right">一个对象中哪些私有信息存放在该对象之外，以及什么时候进行存储</td>
    </tr>
    <tr>
        <td>观察者</td>
        <td align="right">多个对象依赖于另外一个对象，而这些对象又如何保持一致</td>
    </tr>
    <tr>
        <td>状态</td>
        <td align="right">对象的状态</td>
    </tr>
    <tr>
        <td>策略</td>
        <td align="right">算法</td>
    </tr>
    <tr>
        <td>模板方法</td>
        <td align="right">算法中的某些步骤</td>
    </tr>
    <tr>
        <td>访问者</td>
        <td align="right">某些可作用于一个（组）对象上的操作，但不修改这些对象的类</td>
    </tr>
</table>

设计模式之间的关系
![](/assets/images/2020/designpattern.png)

#### 设计模式讨论
**创建型**  
用一个系统创建的对象的类对系统进行参数化有两种方法
1. 生成创建对象的类的子类。缺点在于仅未来改变产品类，就可能需要创建一个新的子类。
2. 依赖于对象复合：定义一个负责明确产品对象的类，并将它作为该系统的参数。

**结构型**  
结构型模式依赖于同一个很小的语言机制集合构造代码和对象
- 单继承和多重继承机制用于基于类的模式
- 对象机制用于对象式模式
- 结构类似的模式
  - Adapter与Bridge常被用于软件生命周期的不同阶段：Bridge在设计类之前实施，Adapter在类设计好后实施。
  - Composite和Decorator都基于递归组合来组织可变数目的对象，通常协同使用。
  - Decorator和Proxy都描述了怎样为对象提供一定程度上的间接引用。

**行为型**  
各个行为模式之间是相互补充和相互加强的关系。

#### 总结
设计模式带来的影响
1. 一套通用的设计词汇
2. 书写文档和学习的辅助手段
3. 现有方法的一种补充
4. 为重构提供了目标