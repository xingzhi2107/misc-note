
# Table of Contents

1.  [继承与原型链](#orgad78617)
        1.  [我的一点个人理解](#org8491f11)
        2.  [原型链的数据结构](#org6d1d1e5)
        3.  [写时“复制”](#org733e4cd)
        4.  [继承](#orga2f704f)
        5.  [实现一个instanceof()](#org5801ec4)
        6.  [obj.constructor.prototype](#org760adc2)
        7.  [Object的API](#org49c78d2)
2.  [词法上下文、运行时上下文与this相关api](#org002d2f6)
3.  [DOM事件模型](#orge45fa45)
4.  [浏览器的Event Loop模型](#org398d1df)
    1.  [重述一遍博客](#orgd926057)
    2.  [javascript的异步编程](#orgca47e82)
    3.  [javascript的Event Loop](#orga510f97)
    4.  [最后](#org15168d3)
5.  [Micro Task与Macro Task](#orgdf1104e)
6.  [JS的GC机制与内存泄露排查](#orgb0d5ede)
    1.  [现存的GC算法](#org58adaac)
        1.  [查找算法](#org91ecdde)
        2.  [GC算法](#org7b8db77)
        3.  [新兴的GC机制——Rust的所有权机制](#org2b4ba71)
    2.  [V8 GC策略](#orgfa313f4)
        1.  [弱分代假设](#org9e3181d)
7.  [Set与Map](#org389b22c)
8.  [JS的深拷贝](#org0c29e5e)
9.  [HTML5 History控制](#orgea54e7e)
10. [单元测试仿真库 &#x2013; sinonjs](#orgb6f2534)
    1.  [Spy](#org7b27061)
    2.  [Stub](#orgf766517)
    3.  [Mock](#org83d1549)
11. [XSS攻击与防御](#org103066a)
    1.  [是什么？](#org0faed03)
    2.  [防御措施](#org280d572)
    3.  [关键](#org59cb287)
12. [CSRF攻击与防御](#org5f012cc)
    1.  [是什么？](#org8bd08ed)
    2.  [防御手段](#org8ad779b)
13. [async与await是如何从generate演化而来的？](#org245407e)
14. [rxjs使用琐记](#orga8a9e4a)
    1.  [Cold的流与性能](#orgb22d677)
    2.  [rxjs具有感染性](#orgf27db32)
15. [Web重绘与重排](#org98d8380)
    1.  [概念](#org25322eb)
    2.  [频繁重绘、重排](#org5b33946)
    3.  [如何避免频繁重绘、重排？](#orgabc6e9a)
    4.  [页面卡顿的原因是什么？](#orgbfd73df)
    5.  [减少卡顿的方法](#org422a089)
16. [SPA的路由问题](#org3ad4eaa)
17. [canvas实现圆锥锥渐变的circle](#orgbd8b2c9)
18. [Egg Language笔记](#org14bd62e)
    1.  [实现一门语言的必要基础](#org7d78ae7)
    2.  [事件循环](#orge111c23)
    3.  [原型链](#org3b18a8f)
    4.  [执行上下文](#org5864f23)
19. [将shell命令行问题转化为js问题](#org2b17b3e)
20. [思考：js的异步解决方案演化——从回调地狱到async](#org0703208)
    1.  [从回调地狱到异步调用链](#orgc99d2b6)
    2.  [从异步调用链到promise](#org0693e26)
21. [getter/setter与IO操作](#org5c32df5)
22. [referer header](#org75e1b7d)
23. [Create Your Own Compiler](#orgc843a08)
24. [chrome devtools](#org2e19bdd)
25. [rxjs性能](#org8848527)
26. [npm包管理](#org4ac02d9)
    1.  [peerDependencies](#org209a482)
27. [零配置的代价](#org5cf6858)
28. [发现mobx与反思](#org5175c60)
29. [proxy get handler的问题](#org38e2855)
30. [JSON.parse BigInt的思考](#org6d9e821)
    1.  [现有解决方案及其问题](#org91d96e4)
    2.  [探索更优雅的方案](#org86cfe9f)
        1.  [溢出不是JSON的问题](#org30c145f)
        2.  [如何与TypeScript和睦相处？](#org16dad7a)
        3.  [总结](#org5c5fe2f)



<a id="orgad78617"></a>

# 继承与原型链


<a id="org8491f11"></a>

### 我的一点个人理解

继承是复用代码的一种常用手段。js的继承机制采用原型链~ 个人感觉这可能仅仅是为了方便，可以直接“草率”的写下一个对象“面量”，而不用Java那样什么都经过精心的绘制对象的蓝图（类），然后才能“实例化”。它的哲学貌似是直接以某个对象为蓝图~ 我不知道这样做是好事坏！对于遍地需要大型前端项目的今天，基于原型的继承貌似有点承受不住了！

JavaScript的继承数据结构——原型链，需要强调的是 **原型** 二字，也就是上面提到的内容，而不是 **链** 这个字。目前我个人所知道的继承机制都是采用“类链式”结构，包括Java。这点很好理解，因为继承是有“先来后到”的，继承经常需要“纠结”的就是顺序问题。链式结构自然的描述了这种先来后到的关系。


<a id="org6d1d1e5"></a>

### 原型链的数据结构

原型链有点像数据结构中的链表，总体而言它包含两部分:

-   **本节点:** 承载属于对象自身的属性、方法
-   **`__proto__`:** 指向自身原型的引用（非公开接口）

当我们对一个对象进行属性访问的时候，如:  ~obj.name~，js的解析器会先从obj的“本节点”中找是否有name这个属性，如果没有则会通过=\_\_proto\_\_=向上查找，直到找到这个属性或者=\_\_proto\_\_=指向null为止。

需要说明的是，=\_\_proto\_\_=不是一个公开接口，在es6中我们可以通过Object.getPrototypeof与Object.setPrototypeof来取代=\_\_proto\_\_=


<a id="org733e4cd"></a>

### 写时“复制”

在JS里，原型只是提供“代码”的复用，也就是“读”。但是在“写”的时候，就没有原型链什么事了。

    var a = Object.create({"name": "hehe"});
    
    a.name;                         // "hehe"
    a.hasOwnProperty("name");       // --> false
    
    a.name = "haha";                // 在“写”的时候，创建a自己的"name"属性
    a.hasOwnProperty("name");       // --> true

JS的这种写时“复制”（新建）的机制，符合原型与实例隔离的需求。


<a id="orga2f704f"></a>

### 继承

最简单的继承方式当然是Object.create()了，这似乎也符合基于原型继承的初衷，但是实际上我们很少这么使用。我们更多的还是像Java那样描述一幅“蓝图”，然后再生产实例&#x2026;&#x2026;(ps. 所以，我们为什么还要用原型继承)。
构造器就是那幅“蓝图”

简单的例子：

    
    // 构造函数用来初始化实例的属性值
    // js是很动态的，所以实际上也可以在构造函数里新建属性，未必需要写到prototype里
    function Animal (weight, age) {
        this.weight = weight;
        this.age = age;
    }
    
    // 定义“原型”也就是“蓝图”
    Animal.prototype = {
        weight: undefined,
        age: undefined,
        display () {
            console.log(`A animal with weight: $(weight), age: $(age)`);
        }
    }
    
    // 定义静态属性。当然了，也可以定义静态方法
    Animal.kindName = "Animal";
    
    let animal1 = new Animal(23, 34);
    
    animal1.weight;                 // 23
    
    // 静态属性不存在于实例上
    animal1.kindName;               // undefined
    
    // 通过new创建的实例都会有一个constructor属性指向构造函数
    // 除非你在prototype里也顶一个了一个constructor把它覆盖了
    animal1.constructor === Animal; // true

接下来开始继承，以Cat继承自Animal为例:

    function Cat(weight, age, color) {
        Animal.call(this, weight, age); // 调用父类来帮助我们初始化属性
        this.color = color;
    }
    
    // Cat的原型继承自Animal的原型
    Cat.prototype = Object.create(Animal.prototype, {
        color: {
            value: null,
            enumerable: true,
            configurable: true,
            writable: true
        },
        catchMouse: {
            // props 描述省略
            value: function() {
                console.log('我在抓老鼠');
            }
        }
    });
    
    // 将Cat的prototype的constructor重新指向Cat
    Cat.prototype.constructor = Cat;
    
    var cat = new Cat(23, 43, 'red');

继承的关键在于指向好原型链，通过Object.create来创建并隔离父类的原型而不是直接让子类的prototype等于父类的prototype，这样做不到修改隔离。


<a id="org5801ec4"></a>

### 实现一个instanceof()

对于=a instanceof B=, instanceof这个操作符的本质是，确认B.prototype在不在a的原型链上。可以说，这个操作符跟=new=是配套使用的。

    function myInstanceOf(obj, constructor) {
        while(obj) {
            if (obj === constructor.prototype) {
                return true;
            } else {
                obj = Object.getPrototypeOf(obj);
            }
        }
    
        return false;
    }


<a id="org760adc2"></a>

### obj.constructor.prototype

今天又手写了一遍继承之后发现，貌似ins.constructor.prototype也可以获取对象的原型，为什么之前没有注意？其实我以前用过这种方法，再模拟es6 class实现一个vue实例的super utils的时候。

但是现在回头想想，这种获取原型的方式是有局限性的。new出来的instance的确可以这样做，但是Object.create出来的对象呢？

所以，es6下获取原型的最保险的方式还是通过Object.getPrototypeOf


<a id="org49c78d2"></a>

### Object的API

js中，几乎所有的对象都继承自Object，所以Object的API几乎适用于所有对象

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />

<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<thead>
<tr>
<th scope="col" class="org-left">分类</th>
<th scope="col" class="org-left">子类</th>
<th scope="col" class="org-left">API</th>
<th scope="col" class="org-left">作用</th>
</tr>
</thead>

<tbody>
<tr>
<td class="org-left">&#xa0;</td>
<td class="org-left">&#xa0;</td>
<td class="org-left">Object.getPrototypeOf(obj)</td>
<td class="org-left">获取obj的原型</td>
</tr>


<tr>
<td class="org-left">原型操作</td>
<td class="org-left">&#xa0;</td>
<td class="org-left">Object.setPrototypeOf(obj, protoObj)</td>
<td class="org-left">指定obj的原型</td>
</tr>


<tr>
<td class="org-left">&#xa0;</td>
<td class="org-left">&#xa0;</td>
<td class="org-left">Object.create(protoObj [,propritesDefineDesc])</td>
<td class="org-left">以protoObj为原型创建对象</td>
</tr>
</tbody>

<tbody>
<tr>
<td class="org-left">&#xa0;</td>
<td class="org-left">self</td>
<td class="org-left">obj.hasOwnproperty(propName)</td>
<td class="org-left">&#xa0;</td>
</tr>


<tr>
<td class="org-left">&#xa0;</td>
<td class="org-left">self, enumerable</td>
<td class="org-left">Object.keys(obj)</td>
<td class="org-left">&#xa0;</td>
</tr>


<tr>
<td class="org-left">属性读取</td>
<td class="org-left">self, enumerable</td>
<td class="org-left">Object.values(obj)</td>
<td class="org-left">&#xa0;</td>
</tr>


<tr>
<td class="org-left">&#xa0;</td>
<td class="org-left">self, enumerable</td>
<td class="org-left">Object.entries(obj)</td>
<td class="org-left">&#xa0;</td>
</tr>


<tr>
<td class="org-left">&#xa0;</td>
<td class="org-left">self, enumerable</td>
<td class="org-left">Object.assign(target, src1, src2&#x2026;)</td>
<td class="org-left">&#xa0;</td>
</tr>


<tr>
<td class="org-left">&#xa0;</td>
<td class="org-left">enumerable</td>
<td class="org-left">for&#x2026;in</td>
<td class="org-left">&#xa0;</td>
</tr>


<tr>
<td class="org-left">&#xa0;</td>
<td class="org-left">self, no Symbols</td>
<td class="org-left">Object.getOwnPropertyNames(obj)</td>
<td class="org-left">&#xa0;</td>
</tr>


<tr>
<td class="org-left">&#xa0;</td>
<td class="org-left">self, symbols</td>
<td class="org-left">Object.getOwnPropertySymbols(obj)</td>
<td class="org-left">&#xa0;</td>
</tr>
</tbody>

<tbody>
<tr>
<td class="org-left">属性创建</td>
<td class="org-left">&#xa0;</td>
<td class="org-left">Object.defineProperty(obj, propName, defineDesc)</td>
<td class="org-left">&#xa0;</td>
</tr>


<tr>
<td class="org-left">&#xa0;</td>
<td class="org-left">&#xa0;</td>
<td class="org-left">Object.defineProperties(obj, defineDescs)</td>
<td class="org-left">&#xa0;</td>
</tr>
</tbody>

<tbody>
<tr>
<td class="org-left">属性描述</td>
<td class="org-left">&#xa0;</td>
<td class="org-left">Object.getOwnPropertyDescriptor(obj, propName)</td>
<td class="org-left">获取属性的*描述*</td>
</tr>


<tr>
<td class="org-left">&#xa0;</td>
<td class="org-left">&#xa0;</td>
<td class="org-left">Object.getOwnPropertyDescriptors(obj)</td>
<td class="org-left">&#xa0;</td>
</tr>
</tbody>

<tbody>
<tr>
<td class="org-left">对象约束</td>
<td class="org-left">&#xa0;</td>
<td class="org-left">Object.preventExtensions(obj)</td>
<td class="org-left">不能新增属性</td>
</tr>


<tr>
<td class="org-left">&#xa0;</td>
<td class="org-left">&#xa0;</td>
<td class="org-left">Object.seal(obj)</td>
<td class="org-left">不能新增属性，不能改已有属性config</td>
</tr>


<tr>
<td class="org-left">&#xa0;</td>
<td class="org-left">&#xa0;</td>
<td class="org-left">Object.freeze(obj)</td>
<td class="org-left">不能新增/删除属性，不能改已有属性config，不能改原型</td>
</tr>


<tr>
<td class="org-left">&#xa0;</td>
<td class="org-left">&#xa0;</td>
<td class="org-left">Object.isExtensions(obj)</td>
<td class="org-left">&#xa0;</td>
</tr>


<tr>
<td class="org-left">&#xa0;</td>
<td class="org-left">&#xa0;</td>
<td class="org-left">Object.isSeal(obj)</td>
<td class="org-left">&#xa0;</td>
</tr>


<tr>
<td class="org-left">&#xa0;</td>
<td class="org-left">&#xa0;</td>
<td class="org-left">Object.isFreeze(obj)</td>
<td class="org-left">&#xa0;</td>
</tr>
</tbody>
</table>

-   **获取当前对象的原型:** Object.getPrototypeOf()
-   **设置当前对象的原型:** Object.setPrototypeOf()

-   **constructor:** 构造函数
-   **什么是实例属性/方法？:** ClassA.prototype上的属性或方法即实例属性或方法
-   **什么是静态属性/方法？:** ClassA上的属性或方法即静态的


<a id="org002d2f6"></a>

# 词法上下文、运行时上下文与this相关api

词法上下文不需要多做解释了，相关关键字：作用域链、闭包。

运行时上下文就是this所指向的对象~ js采用这种模式或许是为了更好的支持“鸭子模型”，以达到复用代码的目的。
call/apply/bind可以改变函数的运行时上下文。

关于call与apply谁快谁慢，完全不用去记忆跟理解，毫无意义；
箭头函数里的this不会指向执行上下文这件事情，也是一样，它就是一种hardcode，为了方便this穿透！


<a id="orge45fa45"></a>

# DOM事件模型

可以说，这篇[文章](http://www.cnblogs.com/binyong/articles/1750263.html)几乎把DOM的事件模型讲透了！

-   **标准事件流:** document->捕获阶段->目标阶段->冒泡阶段->document
-   **监听器:** addEventListener(eventType, handler, useCapture)、removeEventListener(eventType, handler, useCapture)
-   **两种事件传播类型的应用:** 捕获阶段适合全局类型的控制，冒泡阶段适用于一般使用情况
-   **不要记什么？:** 不要去记一堆如何兼容IE的事件机制


<a id="org398d1df"></a>

# 浏览器的Event Loop模型

(ps.Nodejs的Event Loop与浏览器的有点不一样，这里先只说Nodejs的Event Loop)


<a id="orgd926057"></a>

## 重述一遍博客

下面的内容是对这篇[博客](http://cs.brown.edu/courses/cs196-5/f12/handouts/async.pdf)的重述。

大部分人编程都是从c、python、scheme这类典型的“同步”型语言开始的。对于同步型语言来说，他们要如何实现多任务呢？最简单的方式当然是顺序式逐个执行，这种叫“单线程同步模型”。但是这种模型在强调“响应速度”的场景下，如：用户交互式程序，就很难让人接受。另一个选择就是使用多进程或线程的方式来处理多任务，这种叫“多线程模型”或“多进程模型”。但是这种方式涉及到进程、线程间通信的问题，比较复杂，另外当进程或线程数多到一定程度的时候，CPU的资源很大一部分会被耗在切换进程或线程上。
另一种方案就是“异步模型”，就是程序员自己将多个任务切分成小片，交替进行，只要交替的速度够快，那么每个任务都能得到及时的得到响应了，用户会感觉各个任务是“同时”进行的。有没有觉得这句话很熟悉？是不是感觉有点像CPU在多进程间切换模拟多任务一样？区别在于，一个是由程序员控制，而一个是由操作系统控制。


<a id="orgca47e82"></a>

## javascript的异步编程

js实现“异步模式”的方式就是这样将一个任务拆分成多个小分片。举个简单的例子，一个页面加载的时候需要发送两个请求去请求一些数据，假设这两个任务没有相互依赖，那么用js的编程模式，是如何将它们分成小分片的呢？

我们先来描述一下这两个任务要做的事情:

1.  做一些准备事项
2.  发送ajax请求
3.  获取数据，在做一些事情

我们可以更简单的描述为：

1.  before ajax
2.  send ajax
3.  after response

两个任务都可以简单的描述成上方三个阶段，设任务分别为A、B，那么典型的js执行顺序\*可以\*是：

1.  A :: before ajax
2.  A :: send ajax (i/o操作)
3.  B :: before ajax
4.  B :: send ajax (i/o操作)
5.  B :: after response
6.  A :: after response

这两个任务就这样被分成6个小分片。然而，实际是并不是6个小分片，而是4个。因为2跟4都是i/o操作，相对其它四个而言它们其实是非常非常耗时的。然而，js并不care，因为i/o操作js都转嫁给其它线程（也许是进程）来帮忙完成了。

为什么说1、3、5、6是“小分片”？因为它们被执行完毕所需要的时间其实是非常短的，只要你不作死把它们给block住（死循环、调用block api），对我们而言，这两个任务好像真的是并行的进行。


<a id="orga510f97"></a>

## javascript的Event Loop

js的Event Loop机制主要是有一个类似消息队列的队列在辅助完成的。每当遇到“异步任务”时，如：ajax请求、定时任务，js将这些任务交给负责的线程去执行这些耗时的操作，自己则继续往下执行，直到当前栈空了，也就是当前这个小分片执行完毕了。然后，从队列中取出一个“任务的回调函数”，也就是另一个小分片压入栈中继续执行。那么队列中的“任务回调函数”怎么来的呢？程序员在定义“异步任务”的时候注册好的，当其它线程把相对应的任务执行完毕之后，就会把对应的回调函数推入队列当中。

javascript的Event Loop就是这么一回事了！

这也解释了为什么setTimeout时间为0的时候它的回调函数不是立即执行，因为它还要被推入队列中。甚至它要过很久很久才会被执行到，比如栈里正在执行的函数block住了（这个死循环、或者调用window.alert）。


<a id="org15168d3"></a>

## 最后

我觉得Mozilla的文章质量变得很高的感觉，一篇下来简明扼要，[Concurrency model and Event Loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop)。我决定，把它上面关于web的文章都看一遍，或精读，或泛读。

异步编程、Event Loop都不是JS专属，所以我学习JS的Event Loop的时候，很想知道异步编程的本质是什么？除了异步编程还有哪些编程模式？翻来翻去，我找到了这篇博客，[Introduction to Asynchronous Programming](http://cs.brown.edu/courses/cs196-5/f12/handouts/async.pdf)，但感觉还需要进一步的了解，比如可以去体验一下多线程编程，去更深入的了解“异步模型”底层的实现（文中说到的libevent库）。

JS的Event Loop其实还没有完毕，因为Promise的加入，这个模型貌似被做了一些hardcode，nodejs也有这样的hardcode，需要进一步了解。这个另起篇幅描述吧，没必要放在一起说。


<a id="orgdf1104e"></a>

# Micro Task与Macro Task

有些时候，我们需要让一段代码在下一个event loop任务来临之前“立即执行”~
于是浏览器（不同平台实现方式不一定相同）提供了一个新的任务队列叫“Mirco Task Queue”（微任务队列），之前的任务队列就叫做“Macro Task Queue（宏任务队列）”。

“微”就是指“微小”，而“宏”就是指“宏观”。顾明思意，“Micro Task Queue”其实是对“Macro Task Queue”的一个辅助。当执行栈空的时候——也就是下一个event loop任务来临之前，js引擎会去检查“Mirco Task Queue”是不是为空，如果不为空，则把里面的任务都执行了。直到它空了之后，才会去“Macro Task Queue”取下一个任务压入栈中执行。

目前，nodejs的process.nextTick、promise里的onResolve与onRejct都是走“Micro Task Queue”。process.nextTick我能理解它为什么存在，我不能理解的是为什么promise的onResolve与onRejct要这样设计？这个或许应该去深入思考“回调的处理的方式”这个主题。


<a id="orgb0d5ede"></a>

# JS的GC机制与内存泄露排查


<a id="org58adaac"></a>

## 现存的GC算法


<a id="org91ecdde"></a>

### 查找算法

查找算法用来找出可以被GC的内存块

-   **引用计数:** 简单，但无法解决相互引用问题
-   **根搜索算法:** 从根节点出发进行搜索，不能被搜索到都表示需要被GC，又称标记清除算法。这种算法，在搜索的时候，一般需要让“世界静止”，当然也有一些“剪枝”优化，如增量GC、与运行时并行、多个GC并发且与运行时并行

现在，几乎所有的浏览器都采用根搜索算法来进行GC。


<a id="org7b8db77"></a>

### GC算法

GC算法用来处理如何清除需要被GC的内存块，以及如何管理还有用的内存

-   **复制算法:** 将有用的内存逐一复制到一个新的区域，然后清除原有的区域。缺点：耗内存，也耗时。
-   **清除算法:** 直接将需要被清除的内存块清除。缺点：内存碎片。
-   **压缩算法:** 清除完需要被清除的内存块后，把剩余的内存块向一端移动。缺点：耗时


<a id="org2b4ba71"></a>

### 新兴的GC机制——Rust的所有权机制

简单说下关键字就好了:

-   所有权 owner
-   转移 move
-   借用 borrow
-   可变借用 mut-borrow


<a id="orgfa313f4"></a>

## V8 GC策略

以下笔记来自[解读 V8 GC Log（一）: Node.js 应用背景与 GC 基础知识](https://alinode.aliyun.com/blog/37)的摘要。

另一篇文章：[解读 V8 GC Log（二）: 堆内外内存的划分与 GC 算法](https://alinode.aliyun.com/blog/38)也写得非常的好，这里不多记录，因为些得比较细、比较底层。读了一遍，让我印象最深的是“写屏障”，正是这种机制（不单单如此），让V8可以增量GC。


<a id="org9e3181d"></a>

### 弱分代假设

弱分代假设（The Weak Generational Hypothesis）指的是：

1.  大部分对象死得早
2.  那些死得不早的对象通常倾向于永生

因此与JVM类似，V8也引入了对象分代。主要分“新生代”、“老生代”。简单得讲，就是所有对象一开始都是“新生代”，经过一定次数GC还依然存活的对象就晋升为“老生代”。通过分代对不同的代采用不同的GC策略。简单的说，“老生代”的GC频率比较低。


<a id="org389b22c"></a>

# Set与Map

现在想到使用set/map的一个好处是，它可以按插入的顺序来进行遍历，而object做不到着一点。


<a id="org0c29e5e"></a>

# JS的深拷贝

坑点：

-   循环引用
-   一些类型不会拷贝：函数、正则表达式等，需要手动处理


<a id="orgea54e7e"></a>

# HTML5 History控制

html5提供维护history的api。

第一部分：js控制浏览器的“前进”、“后退” 

-   window.history.back()
-   window.history.forward()
-   window.history.go()

第二部分：js修改浏览记录

-   **window.history.pushState:** 新增一条history记录
-   **window.history.replaceState:** 修改当前histor记录（有些浏览器出于安全考虑，依然会新增一条history记录）

第三部分：history变化的事件

-   **window.onpopstate:** pushState、replaceState不会触发这个事件

我以前一直不知道如何用history api支持path的路由控制。现在算明白了。方式如下：

-   window.onpopstate监听“前进”、“后退”的路由变化方式
-   自己封装函数，当调用pushState的时候，也调用一下路由变化的监听函数。这也是为什么vue-router之类的函数监听不到你自己裸调用的pushState、replaceState的原因。我之前傻傻的，一直卡在这里，希望onpopstate能监听到这两个函数的调用，其实就是自己调用的时候，触发一下onpopstate的handler就好了嘛。


<a id="orgb6f2534"></a>

# 单元测试仿真库 &#x2013; sinonjs

之前用的是testdouble（测试替身），有了上次的经验后，这次使用sinonjs就比较顺畅了。对Fake、Spy、Stub、Mock也有了新的理解。


<a id="org7b27061"></a>

## Spy

Spy是“间谍”的意思，它用来给某个函数加上“监听器”，监听函数被调用了几次、参数是什么、返回值是什么等信息。但是，它 **不会改变** 函数的原本行为。
我们可以用来验证被测函数是否调用了某个依赖函数，以及它以什么参数调用、返回了什么值。以此验证逻辑的正确性。


<a id="orgf766517"></a>

## Stub

   Stub的英文意思是：n. 烟蒂；树桩；vt. 踩熄，连根拔除。这里应该是要按动词的延伸意去理解stub的作用，“连根拔除” —— 它实际上替换掉了原来的函数，
所以，它除了有spy的“监听”功能外，还能伪造函数的返回。
   这样，当我们要比较纯净的测试一个函数的时候，就可以把它的依赖函数都stub掉，这样就能隔离测试的目标了！否则，如果依赖函数都执行，那单元测试岂
不成了集成测试？一个底层的依赖函数挂了，岂不是所有上层函数的单元测试都挂了？再有，有些函数是很难执行的、成本很大的，如数据库ORM的save()、get()
；文件系统的读、写。这些我们都可以直接stub掉。


<a id="org83d1549"></a>

## Mock


<a id="org103066a"></a>

# XSS攻击与防御


<a id="org0faed03"></a>

## 是什么？

Cross Site Scripting，跨站脚本攻击。简单点讲就是让目标网站在渲染的时候执行你注入的JS脚本。
目标网站需要你（用户）提供提供的一段内容——可以通过提交数据、URL参数两种方式，然后根据这段内容进行渲染、或执行一定的逻辑，顺带把你的恶意代码给执行了。


<a id="org280d572"></a>

## 防御措施

应对的方式是那句万变不离的老话：不要相信“用户”的输入！具体的措施挺繁琐的，见此[文章](https://mp.weixin.qq.com/s/6ChuUdOm7vej8vQ3dbC8fw)。


<a id="org59cb287"></a>

## 关键

我觉得构建XSS防御体系的关键在于如何“无痕”处理，即防御XSS的逻辑不要混入到业务逻辑当中，我觉得可以采用拦截器、装饰器之类的手段来处理。


<a id="org5f012cc"></a>

# CSRF攻击与防御


<a id="org8bd08ed"></a>

## 是什么？

Cross Site Request Forgery，跨站请求伪造。必须说明的是，这种攻击是针对浏览器环境下的，而不是API环境。
简而言之，它是利用浏览器可以通过=src属性=、~form表单~发起跨域的GET、POST请求，这类跨域请求会直接带上目标域下的Cookie信息，这意味着攻击者伪造出来的请求将带有用户的验证信息，因为出于安全考虑，我们一般都会把session token存放成一个HTTP Only的Cookie项。


<a id="org8ad779b"></a>

## 防御手段

我们只要能证明一个请求不是来自于跨域的=src属性=、或者~form表单~即可。比如，自定义一个Header，规定合法请求必须带有这个Header，因为那两种伪造请求的方式都是无法自定义header的。但是这种方式已经GG了，据说现在通过浏览器插件+重定向的方式可以自定义Header了，详见这篇[文章](https://www.djangoproject.com/weblog/2011/feb/08/security/)。
目前还有两种比较有效的方式：

1.  服务端渲染场景下，将一个token渲染到form的一个隐藏域中，提交的时候带上。这种请求无法被src或者跨域form表单伪造。
2.  前端Ajax请求场景下，将token存放在Cookie（非HTTP Only）中，然后在发请求的时候取出放到body或者header中。这种目前也无法被伪造。


<a id="org245407e"></a>

# async与await是如何从generate演化而来的？

一下内容太监，标题太大，弃文。

先举一个简单的例子，假设有一个数组：callbacks[cb1, cb2, cb3&#x2026;cbn]，里面都是回调函数。每个回调函数自己又会执行一些异步操作，也就是说会注册这个异步操作的回调函数。碰巧，我们需要cb1的异步操作的回调是cb2、cb2的异步操作的回调是cb3，以此类推，我们应该如何把这些回调函数给链起来呢？

我们先来想想，这些函数的签名大概是怎样的。首先，它们自身是作为回调函数的，所以一般会有个参数才接受异步操作拿回来的结果，设为 **data** 。其次，它们会再执行异步操作，并且注册回调函数，一般而言回调函数需要从外界传入，这样更更灵活，所以还有一个参数用来承接回调函数，为了不跟上面的cb1、cb2混淆，我们设为 \*nextCb\*——下一个回调。那么cb1、cb2&#x2026;cbn的函数签名大概就长这样：

    function cb1(data, nextCb) {
        // 做一些事情
        // 异步操作，注册nextCb
    }

  看到这里，你是不是想到改怎么做了？我们只需要把当前执行的结果作为第一个参数，把下一个回调函数作为第二个参数，传入cb即可，然后循环往复。就像这样：
n = 7

let head = cb1(cb2);

    cb1(cb2(cb3(cb4)));

    let data = null;                // 初始上层没有传参数过来
    let callbacks = [cb1, cb2, cb3 ... cbn];
    callbacks.reduce((prevCb, currCb) => {
    cb1(data, )
    });


<a id="orga8a9e4a"></a>

# rxjs使用琐记


<a id="orgb22d677"></a>

## Cold的流与性能

Cold流的每一次订阅都会从头到尾走一遍整个流的周期，这样是不是很耗性能？


<a id="orgf27db32"></a>

## rxjs具有感染性

一旦使用了rxjs，那么所有的地方都需要使用rxjs。
vue的一个组件，prop是一个Observable，那么用到Observable的地方都需要使用Observable链起来。那么问题来了，如果我们已经很明确了，这个组件的生命周期里，props只会有一种组合值，也就是不会变，这时候还是需要使用流吗？
当然可以不需要，但是因为rxjs的感染性，我们不得不用。


<a id="org98d8380"></a>

# Web重绘与重排


<a id="org25322eb"></a>

## 概念

布局的变更会引起重排，即重新排列；颜色、线框等变更（不涉及大小变化）会引起重绘。当然，重排本身就包含了重绘。


<a id="org5b33946"></a>

## 频繁重绘、重排

js调用css接口的时候，频繁“写、读、写、读”css属性的时候会引起重排或重绘（看读写的是什么属性）。因为你"写"了之后，如果立即去读，那么浏览器就要“真实”的渲染一次以获得其属性变更后的效果。


<a id="orgabc6e9a"></a>

## 如何避免频繁重绘、重排？

避免“写读写读”操作，尽量“写写读读”，将写读分批次分离。进一步讲，还可以通过Animoal的一些接口（比如，下一次渲染周期回调之类的）来分离写读


<a id="orgbfd73df"></a>

## 页面卡顿的原因是什么？

一般页面的刷新频率是60HZ，如果一个周期内不能完成渲染，那么页面就会有所卡顿。


<a id="org422a089"></a>

## 减少卡顿的方法

如果一个周期内的时间比较多的被js的耗时操作给抢占了，可以将这些耗时操作放入web worker来处理，这样让主线程主力负责渲染。


<a id="org3ad4eaa"></a>

# SPA的路由问题

早上fix的一个bug引发我的再次思考。关于SPA的路由问题，拿很经典的搜索页面来说吧。

搜索页面，一般会有很多query参数都对页面的内容有影响。用户选中一些过滤条件的时候，需要做如下事情：

-   更新query
-   发请求
-   显示结果

当query变化的时候（其实指的是用户前进、后退浏览器导航的时候），需要做如下事情：

-   发请求
-   显示结果

乍看上去，这两种case都handle，有可能引起死循环，其实可以把两个case，归一化为第二种case。当用户更改过滤条件的时候，只更新query，不做后续的工作。这时候，只要让query跟page组件的key相关，那么query变化了，router组件会重新挂载一个page组件，自然而然的跟大刷新进入当前页面没有太大区别。如果是vue，设置了keep-alive，就可能要注意内存泄露问题，keep-alive貌似没有自动删除缓存组件、或者提供删除缓存组件的接口。

以上所讲的，都是基于page组件这个前提。那些用在modal框里的list组件，一般情况下是跟query不相挂钩的。


<a id="orgbd8b2c9"></a>

# canvas实现圆锥锥渐变的circle

今天要实现一个类似apple watch圆环的动画效果，大概就是这样的：一个圆弧，从无到有，并且圆弧的颜色一直都是从A到B的渐变色。翻来翻去发现，canvas只有两种渐变，线性渐变跟径向渐变，都不是想要的渐变效果。后来查到要求的这种渐变效果叫，圆锥渐变，相关的名词可以看这篇[博客](https://www.cnblogs.com/coco1s/p/7079529.html)，虽然讲的是css，但是概念一致。那么有没有什么办法hack出圆锥渐变呢？

Google了一番，发现了两种方式：

1.  通过一张放射性渐变的位图作为canvas的fillStyle，需要用到canvas的 `createPattern` 接口；
2.  通过一点一点的画，画的过程中改变颜色来实现，简而言之是手动实现渐变；

由于需要的是一个圆弧动画，而不是静态的圆弧，所以第一种方式我就没尝试，直接转向第二种实现方式，这种方式来源于[StackOverflow](https://stackoverflow.com/questions/30631032/continuous-gradient-along-a-html5-canvas-path)。其最核心的部分，就是那个 =getColor=函数。

我的实现思路如下：

1.  每一帧，从0度开始，向前绘制x度的圆弧
2.  对于每一帧的绘制，从圆弧的开始到结尾，再分割成n段圆弧，每一段取的颜色从 `getColor` 中获取。

具体代码见[Fitness process animation](https://gist.github.com/mistkafka/3752f515f493df4b6f6cbe666962866c)。

遇到的问题。最大的问题还是对canvas的接口一无所知，另外google到的很多canvas代码真是难看得一B，关键还是自己不熟。


<a id="org14bd62e"></a>

# Egg Language笔记

最近在重新看[Eloquent Javascript](https://eloquentjavascript.net/)的[Egg language](https://eloquentjavascript.net/12_language.html)（就是用js实现一门叫做egg的编程语言）。为了跟深入的学习js，我尝试往里面加了一些js拥有的特性，如：异步、原型、执行上下文等。收获还挺多的，这里梳理记录下。

特别说明，这些特性的实现思路都是我自己思考的，跟js的官方实现思路是否一致，还未验证过。所以，不要以为这些实现方式就是主流js解析器的实现方式。


<a id="org7d78ae7"></a>

## 实现一门语言的必要基础

我做下来的体会是，要实现一门语言只要有些编程基础即可。可以绕过一大堆编译原理里的词法分析、文法分析等前置概念。等你觉得有意思，有点感性认识后，再回头啃这些概念倒是不迟。

这里推荐两篇入门材料：

1.  [怎样写一个解析器 &#x2013; 王垠](http://www.yinwang.org/blog-cn/2012/08/01/interpreter)。王垠的这篇博客，思路清晰、非常的循序渐进，很适合入门。
2.  [Project: A Programming Language](https://eloquentjavascript.net/12_language.html)，就是Eloquent Javascript这本书的第二个项目实践。英文书开源免费，网上也有中文翻译版卖。

任意实践上面两篇入门材料后，你基本能对词法上下文、闭包、环境、抽象语法树有概念。

下面的笔记，都是基于材料2的。


<a id="orge111c23"></a>

## 事件循环

事件循环的实现，基本没有什么难点。只要对js的时间循环

思考过程：

1.  先只实现宏观任务队列，再考虑微观任务队列；
2.  原本的run函数，执行完毕就退出了，这里肯定要改成一个循环，那么什么时候退出呢？
3.  当宏观任务队列不为空、或者有异步任务未ready的时候，就一直待在循环里。但是，一直死循环也不行，每次sleep 0.1秒去检查状态吧！

异步操作有很多，网络请求、定时器、读文件操作等等，我们这就只实现定时器这种异步操作。需要特别注意的是，需要了解宏观异步任务要经历：被创建、异步任务被执行、异步任务执行完毕（ready状态）推入异步任务队列这三个过程。

1.  首先，我们需要用一个变量作为计数器，表示未ready的宏观异步任务数量；
2.  其次，我们需要一个数组，承载ready的异步任务；
3.  程序一开始的时候，把程序包装成一个ready的task，推入异步任务队列，然后进入循环；

代码如下：

    let unreadyAsyncTaskCount = 0;
    const MAJOR_TASK_QUEUE = [];
    
    function sleep(second) {
      return new Promise((resolve, reject) => {
        setTimeout(resolve, second * 1000);
      });
    }
    
    async function run(...args) {
      let env = Object.create(topEnv);
      let program = args.join('\n');
    
      MAJOR_TASK_QUEUE.push(() => evaluate(parse(program), env));
    
      while (MAJOR_TASK_QUEUE.length > 0 || unreadyAsyncTaskCount > 0) {
        if (MAJOR_TASK_QUEUE.length) {
          const task = MAJOR_TASK_QUEUE.shift();
          task();
        } else {
          await sleep(0.1);
        }
      }
    }

接下来，再让我们实现egg的=setTimeout=等函数

    topEnv['setTimeout'] = function (ctx, callback, ms) {
      unreadyAsyncTaskCount = unreadyAsyncTaskCount + 1;
      return setTimeout(() => {
        unreadyAsyncTaskCount = unreadyAsyncTaskCount - 1;
        MAJOR_TASK_QUEUE.push(callback);
      }, ms);
    };
    topEnv['clearTimeout'] = function(ctx, timer) {
      unreadyAsyncTaskCount = unreadyAsyncTaskCount - 1;
      clearTimeout(timer)
    };
    topEnv['setInterval'] = function(ctx, callback, interval) {
      unreadyAsyncTaskCount = unreadyAsyncTaskCount + 1;
      return setInterval(() => {
        MAJOR_TASK_QUEUE.push(callback);
      }, interval);
    };
    topEnv['clearInterval'] = function (ctx, timer) {
      unreadyAsyncTaskCount = unreadyAsyncTaskCount - 1;
      clearInterval(timer);
    };

这里，不要把js的setTimetout系列函数跟egg的setTimeout系列函数混了。如果你觉得同名看起来太乱，可以给egg的加前缀。

然后，写段测试代码测试一下：

    // support async api
    run(
      'do(define(cb, fun(print("async print"))), setTimeout(cb, 1000), print("sync print"))'
    ); // --> sync print\nasync print

那么如何实现微任务队列呢？其实也很简单。首先，微任务没有等待ready的过程，所以直接插入微任务队列就好了；其次，只要知道微任务的执行优先级高于宏任务就可以了。

更具体的代码见这两个commit：[egg language support async operator](https://github.com/mistkafka/eloquent_javascript_code/commit/012875984f993df9855f0db0d355dfc859970a96) 跟 [egg language support micro task queue](https://github.com/mistkafka/eloquent_javascript_code/commit/70a2ac9d5beb0154bc5dc0f245d4d89cd127a16f)


<a id="org3b18a8f"></a>

## 原型链

原型链的实现，基本没有什么难点。详见commit：[egg language, support object and prototype chain](https://github.com/mistkafka/eloquent_javascript_code/commit/6cd50cc3af5cfe16680bfbfcefe6a00048c7b50c)


<a id="org5864f23"></a>

## 执行上下文

在实现完原型后，我想实现new跟instanceof这两个语法糖，然后演示一下egg的继承。但是，我突然发现我漏了this。所以决定先把执行上下文机制给建立起来。
我感觉执行上下文机制的实现过程还是比较吃力的，所以这里就多啰嗦一些。

我的思考过程：

1.  执行上下文肯定是一个运行时的概念，所以应从=evaluate=函数入手；
2.  this，本质上可以视作一个特殊的变量，既然是变量，那只要执行前注入到localEnv就好了；
3.  现在已经知道在哪里注入执行上下文了，那么这个执行上下文哪里来呢？
4.  突然想起python的method声明方式，第一个参数始终是self，我突然觉得执行上下文应该在apply的时候，作为第一个参数注入进来。

上面是比较顺利的思考过程，再接下来我就陷入一个困境当中。我们在js中，我们能感受到执行上下文的存在的场景有两个：1) `a.f()` 这种调用；2) `var b = a.bind(c)` 这种绑定上下文的操作。我被前一种场景给难住了，一时不知道怎么实现，因为我一直想通过我前面实现的=objectGet=来实现=a.f()=这种调用，但是实际上=objectGet=只相当于=a.f=，而不能做到=a.f()=。
又过了一天后，重新回来面对这个问题的时候，我决定先跳过=a.f()=这种调用，我相信，只要上下文绑定能实现，那么=a.f()=这种调用也就迎刃而解了。所以，我决定先实现一个bind函数。

bind函数很快实现出来了，接受两个参数，=fn=跟=context=，代码如下：

    topEnv['bind'] = function (fn, context) {
      return function (_, ...args) { // _ 这是原来注入的contenxt，摒弃不要了
        return fn(context, ...args);
      }
    };

这其中出了一些问题。我没有意识到topEnv里的这些build in函数，本质上跟specialForm里的fun的返回值是等价的，所以这些build in函数的第一个参数也都是一个执行上下文&#x2013;它们被执行时的上下文，所以代码变成：

    topEnv['bind'] = function (ctx, fn, context) { // ctx是bind函数的执行上下文，context是要绑定给fn的执行下文
      return function (_, ...args) {
        return fn(context, ...args);
      }
    };

下一段egg代码测试一下结果：

    run(`
    do(
      define(display, fun(print(objectGet(this, "name")))),
    
      define(obj1, objectCreate(null)),
      objectSet(obj1, "name", "kafka"),
    
      define(obj2, objectCreate(null)),
      objectSet(obj2, "name", "kk"),
    
      define(obj1Display, bind(display, obj1)),
      define(obj2Display, bind(display, obj2)),
    
      obj1Display(),
      obj2Display(),
    )
    `);    // --> kafka\nkk

非常完美

实现完bind之后，发现js里=a.f()=这样调用就简单很多。首先，需要再次说明=a.f()=其实是两个动作，我想起之前玩es6的Proxy的时候，=a.f()=这样的调用会触发两个Proxy：getter、apply。所以，我不执着于怎么用=objectGet=来实现，我决定新增一个build in函数叫=objectGetApply=来实现。代码如下：

    topEnv['objectGetApply'] = function (ctx, obj, key, ...args) {
      const fn = obj[key];    // 函数所在的obj就是函数的执行上下文
      return fn(obj, ...args);
    };

代码详见这个commit：[egg language, support execute context](https://github.com/mistkafka/eloquent_javascript_code/commit/0d6ff08597174e3d354490828d5fd23dc5e97c55)

在写笔记的时候，我突然意识到，类似=fn.bind(ctxA).bind(ctxB)=这样的结果是什么？ 我期望的是ctxB绑定到fn上。可事实是，这种期望即办不到也不合理，想想为什么？


<a id="org2b17b3e"></a>

# 将shell命令行问题转化为js问题

今天遇到一个这样的问题：
有四个目录，drawable-hdpi, drawable-xhdpi, drawable-xxhdpi，每个目录下都用100+个文件，现在你只想保留这四个目录中名字为：daily<sub>log</sub><sub>light.png</sub>、daily<sub>log</sub><sub>medium.png</sub>、daily<sub>log</sub><sub>heavy.png</sub>、daily<sub>log</sub><sub>crimescene.png的文件</sub>。

用shell怎么解决？
首先，我想到的是shell的通配符。我知道 `rm !(*hello*)` 表示“删除文件名里不带有hello”的文件，但是怎么表示“不是daily<sub>log</sub><sub>light.png或daily</sub><sub>log</sub><sub>medium.png或</sub>&#x2026;”就不知道如何处理。尝试写了个regex，也还是不work。最后，我索性决定用js来处理这个问题。

用js怎么处理？
以往，我都是直接=node=，然后进入在repl下写js。但是说真的，体验不是很好，而且不利于留存一些脚本。我已经在repl下做过很多次这样的事情了，再也无法忍受这样的体验。我决定变革一下。我很喜欢命令下写临时shell文件并执行的形式，可以按下=Ctrl+x Ctrl+e=试试。这种方式是打开一个临时文件，等你写完保存并退出后，执行这个文件。js是不是也可以这样呢？我决定试试。

新的解决方案。
我按照之前的思路，发现js也是可以做到的。方案如下

1.  建个文件夹，比如=~/bin/=，并讲它添加到PATH环境变量里
2.  建个可执行的js文件，例如=~/bin/tmp.js=。可执行的js需要要求：
    1.  文件可执行，运行=chmod +x ~/bin/tmp.js=即可
    2.  文件头要有这样的声明=#!/usr/bin/env node=，表明使用node来执行这个文件
3.  运行 =vim ~/bin/tmp.js && tmp.js=，就能达到shell的=Ctrl+x Ctrl+e=的效果。写个hello world试试，保存并退出后是不是立即执行了？
4.  再给这个命令加个alias，这样就不用每次都打那么长的命令了。

这样，我就可以很方便的写js，并执行它。随着时间的推移，肯定会积累一些通用、方便的操作符，这样对shell的一些操作，肯定能越来越有效率。

一点小建议。
在执行任何删除、变更命令之前，一定要有一步，确认环节。比如，把要删除的内容打印出来，确认了，再执行真正的删除。另外，其它语言如：ruby、python也可以按照这样的思路来做。总而言之，如果你不是非常精通shell，那么复杂的shell问题，可以转化为你熟悉的语言来解决。

新的发现。
试用了几次，发现这样有时候也不是很爽。有些时候，还是node那种repl的模式比较方便。但是，每次进入repl又要手动load一些东西，不然每次都从头写太浪费时间了。去搜索了一下，找到一个自动load的方案：

    > NODE_PATH=/Users/zhenguo/bin/node_modules node -i -e "$(< ~/bin/utils.js)"

讲解。使用=-i=参数使之强制进入repl模式，而=-e=参数就是执行一段js，再后面的是shell的指令，可以把=~/bin/utils.js=的内容作为输入喂给=node -i -e=。整句话的意思就是，强制进入repl模式，并且在repl下执行=~/bin/utils.js=的内容。这样，你只需要把一些积累下来的工具函数放在那个文件里，每次启动的时候就不需要再手动载入了。至于=NODE<sub>PATH</sub>=，这个环境变量，是为了保证你的=utils.js=能load到js库，具体参见node官方文档。


<a id="org0703208"></a>

# 思考：js的异步解决方案演化——从回调地狱到async

以前看过别人写async是如何实现的之类的文章，不过现在忘了。之前重新思考这个问题的时候，就突然想到为什么不从更“源头”的地方开始想起呢？
不过这个问题搁置了很久，去年重新复习js的时候开始有了这个想法，并开始思考，但是很快被搁置了，直到今年。希望，今年能把这条线路打通。


<a id="orgc99d2b6"></a>

## 从回调地狱到异步调用链

将回调地狱转化为异步调用链的做法，有点像将递归实现改成循环实现。回调地狱大家应该都知道是什么了不再多说，这里就说明一下我所说的“异步调用链”是什么。直接上代码：

    function afterOneSecond(currSecond, cb) {
        setTimeout(() => {
            const second = currSecond + 1;
            console.log(second + 's');
            cb(second);
        }, 1000);
    }
    asyncChain([
        (_, next) => afterOneSecond(0, next),
        (val, next) => afterOneSecond(val, next),
        (val, next) => afterOneSecond(val, next),
        (val, next) => afterOneSecond(val, next),
        (val, next) => afterOneSecond(val, next),
    ])
    
    // result:
    // 1s
    // 2s
    // 3s
    // 4s

在实现出promise之前，我准备先实现出这种粗糙的异步调用链。

让我们开始吧，先来看回调地狱：

    setTimeout(() => {
      console.log('1s');
      setTimeout(() => {
        console.log('2s');
        setTimeout(() => {
          console.log('3s');
          setTimeout(() => {
            console.log('4s');
          }, 1000);
        }, 1000);
      }, 1000);
    }, 1000);

我故意将它写成每一层的操作都很类似，这样容易让我们想到用递归的方式去实现上面的代码：

    function afterOneSecond(currSecond) {
      setTimeout(() => {
        const second = currSecond + 1;
        console.log(second + 's');
    
        if (second < 4) {
          afterOneSecond(second)
        } else {
          // exit recursion
        }
      }, 1000);
    }
    afterOneSecond(0);

你可能会说，实际情况根本不可能让每一个异步调用都类似，从而让你可以用递归来表示。其实问题不打，即便各个异步调用很不一致，我们也可以给每一个异步操作包上一层统一的外衣。这个先搁置不谈，来说说怎么把上面这个递归改实现改成一个循环实现吧！

在这个过程中，我主要遇到了两个问题：

1.  执着于for循环，导致实现起来比较棘手；
2.  没有搞清楚回调地狱的依赖本质，从而一开始搞错遍历的方向；

关于第一个问题。for循环本质可以视为遍历数组，所以只要是数组遍历就ok。总之，不该拘泥于此。
关于第二个问题。可能是上面的那个递归版本的代码给了我误导，让我没有意识到“前一个异步操作，总是依赖后一个异步操作”（递归只要依赖自己就行了），所以总是想前往后遍历，但实际上从后往前遍历会方便很多。

摸爬滚打后，得出实现代码：

    function asyncChain(fns) {
        const headFn = fns.reduceRight((next, fn) => { // 从后往前遍历
            // 将当前异步函数包裹成前一个异步函数的next
            const newNext = (val) => fn(val, next);
            return newNext;
        }, () => {}); // 最后一个函数的next为空
        headFn();
    }
    
    function afterOneSecond(currSecond, cb) {
        setTimeout(() => {
            const second = currSecond + 1;
            console.log(second + 's');
            cb(second);
        }, 1000);
    }
    asyncChain([
        (_, next) => afterOneSecond(0, next),
        (val, next) => afterOneSecond(val, next),
        (val, next) => afterOneSecond(val, next),
        (val, next) => afterOneSecond(val, next),
        (val, next) => afterOneSecond(val, next),
    ])

在这里，其实我已经同一化了异步函数的接口，也就是我上面说的“包上一层统一的外衣”。

异步调用链，铺平了回调地狱，并且统一异步调用之间的传值方式。

那么铺平的异步调用链，如何向promise演进呢？先看看promise做了什么事情？

1.  铺平异步调用；
2.  向外暴露了异步状态，有点发布订阅者的感觉；
3.  统一异步与异步之间的传值、异常传递；

我决定先实现一个只做到第一点的简易promise。


<a id="org0693e26"></a>

## 从异步调用链到promise


<a id="org5c32df5"></a>

# getter/setter与IO操作

今天要写一个跑在浏览器端的脚本，因为需要reload，所以只能通过localStore来存储状态。

不停的getItem，setItem让我觉得有点烦了。于是，我用Proxy，实现了一个可以点操作的store，代码如下：

    const store = new Proxy({
      task: null,
      keywords: [],
      links: [],
      walkedLinks: [],
      products: [],
      currentKeyword: null
    }, {
      get(target, key, receiver) {
        const itemKey = 'GLOW_KEY_' + key;
        const strValue = localStorage.getItem(itemKey);
        if (strValue) {
          return JSON.parse(strValue);
        } else {
          return Reflect.get(target, key, receiver);
        }
      },
      set(target, key, value, receiver) {
        const itemKey = 'GLOW_KEY_' + key;
        const strValue = JSON.stringify(value);
        localStorage.setItem(itemKey, strValue);
      }
    });

这样用的时候似乎是方便了一些，但其实隐患极大。

1.  `store.products` 这句话每次执行getter操作的时候，都会产生一个新的products对象。是每一次噢！不要以为store是个普通的对象。
2.  `store.products.push(x)` 并不会自动同步到localStoreage。原因同第一点一样，因为它是一个全新的对象。
3.  不管是setter还是getteer操作，每次都伴随着一次I/O操作，当然大概率下不会有大问题。

我使用完之后觉得，I/O操作不适合封装成这种隐式的getter/setter操作，而是应该：

1.  采用函数调用，这样很容易知道你获得的是一个副本，并且函数相对于getter/setter会让人更注意操作是不是I/O操作；
2.  或者，像很多ORM框架一样，采用所谓的instance缓存起来，需要写回storage的时候，显示的调用commit()。这样虽然依然使用getter/setter，但是因为commit的存在，开发者就会注意到那是I/O操作，又因为是缓存起来的，所以多次的getter操作拿到的对象也是同一个对象。


<a id="org75e1b7d"></a>

# referer header

发现已经将近一年处于游荡状态了，好久不记录一些小知识。
今天仔细学习下referer header。

场景是这样的：
有个前端跳转页面，还有个后端跳转，这两个跳转的target<sub>url都是可以让输入者随便填的</sub>，这样就可以让攻击者把target<sub>url指向他们的挂马网站</sub>。一般而言，他们的挂马网站会被各大浏览器、网站加入黑名单的，但是通过我们的域名他们可以绕过这个黑名单。我感觉应该是一定程度上，不一定就都能绕过，不过这个概率不重要，我们要做的是尽可能把这个概率降低。我们想探索一下，是不是只要验证referer header就可以了。

查阅MDN还是欠点感觉，看完[这篇](http://www.ruanyifeng.com/blog/2019/06/http-referer.html)blog就差不多都了解了。类似图片防止外站访问，我们可以在server端、client端都检查一下referrer，只要不是自己的网址，都block住。


<a id="orgc843a08"></a>

# Create Your Own Compiler

[Create Your Own Compiler](https://citw.dev/tutorial/create-your-own-compiler?p=5)在线的教程


<a id="org2e19bdd"></a>

# chrome devtools

这个系列的文章还不错，[Link](https://zhuanlan.zhihu.com/p/80792297)。


<a id="org8848527"></a>

# rxjs性能

用rxjs改造完entity之后，内存从20mb飙升到500mb，简直吓我一大跳。
[RxJS Patterns: Efficiency and Performance](https://blog.bitsrc.io/rxjs-patterns-efficiency-and-performance-10bbf272c3fc)，这篇文章不错。我觉得最大的问题应该是pipe里发生了re-subscribe，一般的sub/unsub我都清理得很好的。


<a id="org4ac02d9"></a>

# npm包管理


<a id="org209a482"></a>

## peerDependencies

[这篇](https://www.cnblogs.com/wonyun/p/9692476.html)文章讲得非常不错，另外看了npm官方的[文档](https://nodejs.org/es/blog/npm/peer-dependencies/)，peerDependencies不仅仅是为了让host project可以import子依赖，还是为了较少同一个依赖安装多次。
之前把一个项目的通用组件抽离出来共享的时候，就出现过因为react-native的版本不一致，导致安装的时候安装了两个rn包。后来把两边的rn版本统一了
之后，安装速度明显快了。这个应该是yarn/npm自带的功能，合并同一个依赖范围的依赖，以后可以研究下。如果之前一开始在声明通用组件包的时候，就
把react跟rn声明在peerDependencies里，这样顶层项目安装的时候就会因为版本同一致而报warning。

另外，不得不说peerDependencies做得还是有问题，当多个package的peerDependencies相互冲突的时候，为之奈何？现在项目经常看到一大片的peer
dependencies warning。


<a id="org5cf6858"></a>

# 零配置的代价

经过了半天的“试错”，总算把react-native-web的storybook配置好了。怎么好的？我也不知道。react跟storybook都试图提供一种另配置的项目初始化方案，希望
一个简单的=init=就能解决问题。这样的愿景固然是好的，但是现实的情况却是复杂的。storybook并没有针对react-native-web的preset，我参照一些blog的方法
一步一步配置好之后，得到的却是各种load、babel、modules not found之类的错误。

这时候我该怎么办？只把配置的options都读一遍，看看是不是哪里多了或者少了。但是storybook的配置项几乎为零，试来试去都不成功，简直好无头绪。最后，我
是通过在一个完全新的项目里，再把之前的几种试错方案又试了一遍，才找到一种成功的配置方案。那么为什么原来的项目里不work？不知道。真正的原因埋在一通
乱七八糟的报错信息背后。

一个工具提供零配置方案固然好，但是代价就是一旦出了问题，我们就不知所措。我感觉好的工具，最好是提供几个简单的概念（接口），然后通过这些概念进行编程
组合。以build工具为例，我更喜欢gulp这类编程风格的工具，而不是webpack这种配置项风格的工具。

我感觉对我而言，终极是要打开build工具的黑盒。

继续昨天的吐槽。虽然不喜欢webpack，但是相比较create-react-app这类对webpack二次封装之后的工具，我感觉还是直接用webpack比较好。正如昨天所想的那样，这个
黑盒终极是要打开，要去熟悉webpack的机制，常用的插件，甚至要打开webpack的源码。或许这样之后，能弄出一个编程配置风格的webpack。所以，从现在开始维护自己
的webpack template吧！


<a id="org5175c60"></a>

# 发现mobx与反思

redux这个东西真的是难用得不行，现在一写到要跟redux打交道得代码就很烦躁。

-   没有computed机制，还要套一个selectjs才能实现。
-   只能订阅整棵状态树。这意味着性能差，并且导致selectjs的写法非常的傻缺。

上网吐槽之际，才去想着看看别的解决方案，才发现mobx就很完美嘛（目前而言）！

不过重点不是发现mobx，重点是为什么看过那么多次mobx这个词，自己就没想着稍微深入的了解一下呢？那种“对框架疲惫”的心态真的是毒瘤！勇于探索新事物，尝试新事物，面对槽点勇于发声，不然类似的事情会在发生，自己离被淘汰也就不远了。


<a id="org38e2855"></a>

# proxy get handler的问题

先前，通过proxy在RN的StyleSheet基础上封装了一个比较无感知切换color schema的StyleSheet工具。其原理，其实就是用proxy get handler来拦截styles的getter操作，在内部动态判断当前应该用什么color schema。
今天有同事跟我说，把整个styles传入一个第三方组件的时候，完全没有起到作用。我debug后，发现这个第三方组件把传入的styles，用 `...` 解构了一遍。自己实验了一下，才发现styles被 `...` 解构的时候，是个空对象。这是因为，在用proxy封装的时候，target对象本身就是一个空对象。js解析器当然不知道它有什么key了，解构成空对象也很合理。

    new Proxy({}, styleProxyHandler);

经过尝试，发现加个ownKeys handler也是没用了。最终是把对象的key，全部复制一份到target里。

    // need a "empty" target with all keys
    // to fix {...obj} operator
    const emptyTarget = Object.keys(lightStyles).reduce((obj, key) => {
      obj[key] = null;
    
      return obj;
    }, {} as any);
    
    return new Proxy(emptyTarget, styleProxyHandler);


<a id="org6d9e821"></a>

# JSON.parse BigInt的思考

今天又有一位客串写JS的同事被js的整数溢出坑了一下午，让我不禁又要重新思考这个老问题。


<a id="org91d96e4"></a>

## 现有解决方案及其问题

这个问题一般出现在server跟client端通讯的情况。之前的解决方案是在server端做转换。即www层有个拦截器，遍历整个object，在返回数据的时候把以id结尾的key的值转成string，或者接收client数据的时候把string转成int。

这个方案算是基本可行。但还是存在问题：

-   严重依赖字段的命名，字段如果不是以id结尾就不起作用。
-   虽然可以继续加这个关键字列表，但是还是很粗糙，而且可能“误杀”。
-   从源头开始就“污染”了类型。本该是Int的值，从此都必须是string

Twitter的方案。跟上面的方案大同小异，他们的方案就是给会溢出的字段多增加是str字段。如id会溢出，那就增加一个id<sub>str字段</sub>。

这种int转string的方案虽然丑陋，但是还算“稳定”。一般而言，我们也不会拿id去做一些减价乘除的运算，它说白了就是一个符号，在client端用string承载，也还合理。

但是，有没有更优雅的方案？


<a id="org86cfe9f"></a>

## 探索更优雅的方案


<a id="org30c145f"></a>

### 溢出不是JSON的问题

int溢出是JS的问题，在client端拿到JSON数据的时候，int的信息还是存在的。只不过从JSON解析成JS的value的时候，溢出了。通过一些小技巧，可以防止int溢出。

-   通过reviver参数，详见[这篇回答](https://stackoverflow.com/a/69644630)。总结一下就是对JSON预处理，把会溢出的int变成string。然后在parse的时候，把这种string转成BigInt。
-   使用[json-bigint](https://github.com/sidorares/json-bigint)库。看了下它的[源码](https://github.com/sidorares/json-bigint/blob/master/lib/parse.js)，大概是用js实现一个JSON解析器。

有了JSON与JS相互转换不丢失信息的方案，就可以不在server端做转换。


<a id="org16dad7a"></a>

### 如何与TypeScript和睦相处？

虽然有了可以在client端做转换的方案。但是新的问题也出现了：一个int值，什么时候会被转成BigInt，什么时候会被转成number？假设，有个topic实体，它有一个id字段。这个id字段可能溢出。

大部分情况下，都不用关心这个问题。因为我们基本不会拿id去做一些运算，这种情况下建议把id标记为 `number | BigInt` 类型，这样遇到有问题的场景的时候，期待TypeScript可以提醒我们。

另外，也可以图方便。直接让所有的int都转成BigInt，所有的这些字段都标记为 =BigInt=。但是这样做会不会有性能问题？有待考究。


<a id="org5c5fe2f"></a>

### 总结

综上有个大概得方案以待验证：

-   使用json-bigint库，在api层把JSON文本转成object以防止int溢出。配置所有的int都转成BigInt。
-   所有的int字段类型都从number改成BigInt。

这样的做法会不会更优雅？

