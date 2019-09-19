# *Refactoring* 阅读笔记

[Refactoring: improving the design of existing code (2nd Edition)](https://refactoring.com/)

<p align="center">
  <img width="220" height="300" src="https://refactoring.com/refact2.jpg">
</p>

有条件者，推荐购买 [中文实体书籍](https://item.jd.com/12584498.html) 或电子版进行阅读。本书第二版使用了 `javascript` 进行编写，而重构手法与语言无关（第一版为 `java`）。此文档仅为个人笔记，不能包含所有内容（手法），且带有个人意见的附注和修改。本文档会有未记载的手法，如果需要的话，直接阅读原文会比较好。

本书中的「重构」一词表示一种特定的清理代码的方式，不会改变软件可观察的行为。这意味着，**严格按照重构手法修改的代码很少进入不可工作的状态**，即便重构没有完成，也（几乎）可以随时随地停下来。如果有人说他们的代码在重构时有一两天时间不可用，基本上可以确定他们做的事不是重构，那是重写。（p.46）

重构不应对原行为产生影响，如果已经存在被观测到的 bug ，则使用下列手法重构结束后其依然要存在；如果没有还产生影响，也可以顺手修复它们。

阅读笔记的记录格式：
- 名称
- *简介（如果无法使用 200 字阐述，则阅读毫无意义）
- 代码展示（摘于 [官网目录](https://refactoring.com/catalog/) ）
- 动机（为什么要做这个重构 & 什么情况下不该做这个重构）
- 步骤（实现此重构遵循的流程）
- *笔记（其他思考）

使用 [GayHub](https://github.com/jawil/GayHub) 提高阅读体验

---

## 目录
- [*Refactoring* 阅读笔记](#refactoring-阅读笔记)
  - [目录](#目录)
  - [书中部分术语的提前声明](#书中部分术语的提前声明)
    - [命令与查询](#命令与查询)
    - [可变数据与不可变数据](#可变数据与不可变数据)
  - [何时重构 p.50](#何时重构-p50)
  - [重构与性能 p.65](#重构与性能-p65)
  - [代码的坏味道 p.71](#代码的坏味道-p71)
  - [构筑测试体系 p.85](#构筑测试体系-p85)
  - [第一组重构 p.106](#第一组重构-p106)
    - [提炼函数（Extract Function）](#提炼函数extract-function)
    - [内联函数（Inline Function）](#内联函数inline-function)
    - [提炼变量（Extract Variable）](#提炼变量extract-variable)
    - [内联变量（Inline Variable）](#内联变量inline-variable)

<br />

## 书中部分术语的提前声明

### 命令与查询

- 命令：带有副作用（与非局部变量进行交互，如 IO 、修改全局变量）的函数
- 查询：带有返回值的函数

一般来说，应该保持命令与查询的分离。查询保持纯函数的性质，不论多少次调用都不会产生影响。

### 可变数据与不可变数据

- 可变数据：可以赋值和读取的数据（字段）
- 不可变数据：只可以读取的数据（字段）

可变数据意味着，一个实例的某一字段被暴露出来了，从而可以用赋值语句修改它。然而，不可变的数据是强大的代码防腐剂，它意为着你想要改变其内部时，只能通过创建新的数据来完成。不要对 `new` 抱有性能焦虑（见 [重构与性能](#重构与性能-p65)）。如果数据不可以修改，意味着你可以放心大胆地使用，而不用担心它会有其他逻辑导致内部发生改变。

<br />

## 何时重构 p.50

- 预备性重构：使添加功能更容易。「如果对现有代码库进行微调，会使修复 bug 或添加新功能更容易」
- 帮助理解的重构：使代码更易懂。「无法理解现有代码，或理解起来又困难的时候，可以进行重构」
- 捡垃圾式的重构：我理解代码在干什么，但发现它做的不够好。
- 有计划的重构和见机行事的重构：一般不会给重构留出单独的时间，应该在做其他事的时候自然而然地发生重构。上面提到的都是见机行事的重构。

注意：每次要修改时，首先要使修改变得容易（p.53）。因而下面的部分重构手法（大多数在「第一组重构」中）是很重要的，并且其他一些重构会因准备工作而显得比较迂回。

<br />

## 重构与性能 p.65

- 对于实时系统（心率调节器等），需要提前给各个组件做好时间估算，并持续关注它们的性能变化；
- 而对于其他绝大多数系统而言（诸如用户界面系统、企业信息系统），这样就会大动干戈了。实际上，如果对绝大多数程序进行分析，你会发现它的大部分时间用在了一小部分代码上，这意味着常规的 90% 的优化工作是白费劲的（因为那些优化部分很少被执行）。一种合理的编程手段是，直到发生了可观测的性能问题，再使用性能优化手段进行调优；否则，我们认为重构不影响系统的性能。这一般很难被接受，但你应该知道的是，重构后的代码更易于找出性能热点，也更易于展开优化。

重构与性能优化有很多相似之处，但他们不一样：

- 重构为了让代码更容易理解，更易于修改，有可能导致程序更快，也可能更慢；
- 性能优化只关心让程序运行地更快，但最终的代码可能更难以理解和维护（要有心理准备）。

<br />

## 代码的坏味道 p.71

什么时候应该开展重构？代码有坏味道的时候。虽然下面的坏味道描述尽管很抽象，但不妨可以看一看。

- 神秘命名（Mysterious Name）
  - 命名是编程中最难的事之一（无头绪的时候，可以到 [codeif](https://unbug.github.io/codelf/) 去查翻译。有趣的是，网站的那句话就是从本书里摘抄的）。当出现不好的名字时，往往以为着更深层次的设计问题（如一个函数做了太多事情）。这时候可能会需要 `变量改名` 、`字段改名`。
- 重复代码（Duplicated Code）
  - 事不过三，三则重构。(使用 `提炼函数`)
- 过长函数（Long Function）
  - 百分之九十的场合里，把函数变短只需要展开 `提炼函数`。每当你觉得需要注释来说明什么的时候，就是提炼函数的最好时候。函数名？浓缩一下写出来的注释就有了。
- 过长参数列表（Long Parameter List）
  - 有几个参数总是一同出现，则可以使用 `引入参数对象`。
  - 某个参数其实可以向某一实例发起查询而获得，那么就应该在函数里进行查询而非传参（`以查询取代参数`）
  - 某个参数用于区分函数的行为（函数内只是用于 `if` `else`），则使用 `移除标记参数`。
- 全局变量（Global Data）
  - 除非在用 C语言 写操作系统，否则你应该使用 `封装变量` 进行重构。
- 可变数据（Mutable Data）
  - 使用 `封装变量` 确保所有数据更新都只通过几个函数来进行。
- 发散式变化（Divergent Change）
  - 最好的编程情况是，如果需要修改已存在的模块的行为，可以直接跳到它的某一点，只在该处进行修改。否则，这就是发散式变化。「每次只关心一个上下文」很重要，你可能 `搬移函数` 或者 `提炼类` 等进行修改。
- 霰弹式修改（Shotgun Surgery）
  - 类似于发散式修改，但有所区别。霰弹式修改意味着每遇到一种变化，就需要在许多不同的模块中做出许多小修改。这种情况下，应该使用 `搬移函数` 或 `函数组合成类` 等进行修改。
- ...... 坏味道还有很多，但味道这东西需要你自己闻出来。你越熟悉各种重构手法，就会在代码里闻出越多的坏味道。所以，「talk is cheap, show me the code」！

<br />

## 构筑测试体系 p.85

重构很有价值，但只有重构还不行。要正确地使用重构，还得有一套牢固的测试集合，以帮助发现难以避免的疏漏。一个坚固的系统，应该使用一套合理的测试框架（如 GTest，JUnit 等）构建一些自动化测试，让所有对系统的修改都能通过测试。

现代编程语言都有测试框架。唯一阻止你进行测试的原因只有没时间写测试代码。写出好的测试框架并且时常运行它们，可以让开发时有足够的信心推进工作。

注意：
- 你应该总是确保测试不该通过时真的会失败，因而添加几个错误输入时的测试会很有帮助。
- 每当你收到一个 bug 报告，请先写一个单元测试来暴露这个 bug。

写多少测试才够？如果只是个人项目，这个问题没有标准答案。因为这需要你自己根据时间、人力来衡量重构和测试的细节。单元测试是最容易编写的，每当新增一个功能，你可以先编写对应的单元测试。

<br />

## 第一组重构 p.106

### 提炼函数（Extract Function）

**简介**

一个函数可以通过调用其他函数来完成任务，最好不要在其内配置子过程来完成任务。子过程越多，代码越长，会导致代码的可复用性、鲁棒性、可阅读性越差。调用函数时，函数名可以透露操作目的，减轻阅读时的心理负担。

**代码展示**

```javascript
//old
function printOwing(invoice) {
  printBanner();
  let outstanding  = calculateOutstanding();

  //print details
  console.log(`name: ${invoice.customer}`);
  console.log(`amount: ${outstanding}`);  
}

//new
function printOwing(invoice) {
  printBanner();
  let outstanding  = calculateOutstanding();
  printDetails(outstanding);

  function printDetails(outstanding) {
    console.log(`name: ${invoice.customer}`);
    console.log(`amount: ${outstanding}`);
  }
}
```

**动机**

- 作为其他手法开展前的准备工作（便于成块地调成目标代码）
- 出现过多个类似的代码段
- 需要花时间才能弄明白函数内的某一段代码在干什么
- 函数承载了太多的功能/步骤（两个？三个？依情况而定）
- 函数内有需要注释的代码段
- 函数太长（作者认为超过6行的代码就是在散发臭味）

**步骤**

- 创建新的空函数，根据意图/行为来命名
  - 从其他函数内抽取出来的代码段，如果原来有注释，从中提炼名称。
  - 如果没有，先写注释（在脑子里也可以），然后使用上面的方法。
  - 需要翻译时，使用 [codeif](https://unbug.github.io/codelf/)。
  - 以做什么来命名，而非怎么做。
- 复制（而不是剪切）源代码到目标函数
- 检查新代码，是否有无法接触的变量（实现功能的代码段一定用到了变量）
  - 如果是支持函数嵌套的语言且只提炼到内部，则不存在此问题。
  - 如果提炼到了外部，则将变量作为参数传入。
  - 如果变量在代码里被赋值或操作，要小心：考虑将传入其引用/指针或者尝试构建查询函数，在原代码中先使用查询为其赋值。
  - 如果参数太多，考虑 `引入参数对象` 或者 `以查询取代临时参数`，甚至停止重构。
- 编译，使新函数能否通过
- 在原函数中将目标代码段替换为新函数调用
- 测试

**笔记**

第一个重构手法，也是最重要的重构手法。在这里，你应该停留足够多的时间来尽量理解此手法，因为其他手法经常需要此作为基础工作。更如果无法理解也没关系，略读一遍本书后再回头看，你会大有收获。

值得注意的是，本书中对于代码变动十分谨慎，力求重构过程不破坏原代码，因而其中一步使用了复制而不是剪切。初读本书，一定会无法理解为何使用如此细碎的脚步。但到后期，手法和代码愈发复杂时，不破坏的重要性就会显现出来。如果重构过程中止，你完全可以直接删除重构部分（或者使用 `git stash`，但这可能需要在重构前 commit 一下），从而恢复现场。

重构不是「银弹」，在重构过程中也许会觉得重构会带来更大的麻烦而决定中止。但一般情况下，导致重构手法「ugly」的根本原因是设计或者其他层面的问题。这时候，你可以选择先使用对应的重构手法来进行提前工作。

### 内联函数（Inline Function）

**简介**

有时候会有一些函数，它们简短地就如其内部代码一样易读，且只有其他某一个地方调用到了。这也许是过度使用 `提炼函数` 造成的。此时，把它们重新内联回去的尝试是有益的：熟练使用自动化重构工具或者其他辅助工具（如自己写的 [snippets](https://code.visualstudio.com/docs/editor/userdefinedsnippets)）时，这样的动作只耗费几分钟，而你对代码的理解会更进一步。

**代码展示**

```javascript
//old
function getRating(driver) {
  return moreThanFiveLateDeliveries(driver) ? 2 : 1;
} // 代码展示中的名字不重要，重要的是它们的关系

function moreThanFiveLateDeliveries(driver) {
  return driver.numberOfLateDeliveries > 5;
}

//new
function getRating(driver) {
  return (driver.numberOfLateDeliveries > 5) ? 2 : 1;
}
```

**动机**

- 一个函数调用了许多不甚合理的小函数，现在希望重新整理它们
- 代码中有太多中间层，使得系统的所有函数就像只是对另一个函数的简单委托，现在希望去掉无用的中间层

**步骤**

- 检查函数，确定其不具有多态性（多态性意味着此函数是「virtual」的，调用其可能会转而调用子类的同名函数。如果是这样，那么无法内联）
- 找出所有调用点（一般只有一两处地方调用才想内联，否则还是保持其现状才好）
- 将调用点替换为函数本体
- 测试
- 删除函数定义

**笔记**

这样看来，内联函数可能很简单。但其实不然，你会面临自己内心的「以后会不会需要它的存在」的拷问。这样的担心其实是多余的，因为当以后需要重新提炼它的时候，你完全可以返回重构之前的版本复制函数回去（每一个重构手法，都进行一次 commit，以便回退时不会混乱）。


### 提炼变量（Extract Variable）

**简介**

过长的算术表达式和过深的嵌套（大于三层）会导致难以阅读。由于重构时认为变动所导致的性能损失为 0，你大可以为了阅读而不顾 O(1) 的性能损失。

> 傻瓜都能写出计算机可以理解的代码，唯有能写出人类容易理解的代码的，才是优秀的程序员。—— p.10

**代码展示**

```javascript
//old
return order.quantity * order.itemPrice -
  Math.max(0, order.quantity - 500) * order.itemPrice * 0.05 +
  Math.min(order.quantity * order.itemPrice * 0.1, 100);

//new
const basePrice = order.quantity * order.itemPrice;
const quantityDiscount = Math.max(0, order.quantity - 500) * order.itemPrice * 0.05;
const shipping = Math.min(basePrice * 0.1, 100);
return basePrice - quantityDiscount + shipping;
```

**动机**

- 语句执行的内容过多导致调试困难
- 难以理解该语句的作用

**步骤**

- 确认提炼不会有副作用（一个赋值语句不应该带副作用，否则就太「ugly」了）
- 声明一个变量容纳想要提炼的表达式。如果想不出，可以使用临时的变量名（如 `xxx` 或 `zzz` 前缀方便搜索），或者到 [codeif](https://unbug.github.io/codelf/) 上去查询
- 用新变量替换原来的表达式
- 测试

**笔记**

变量命名是每个程序员的噩梦。但是，你完全可以写一段注释然后从中提取关键词来作为名称。


### 内联变量（Inline Variable）

**简介**

有时候，过度地提炼变量会妨碍后来的重构和阅读。如果一个变量不比表达式有表现力，那么你应该将其内联回来。

**代码展示**

```javascript
//old
let basePrice = anOrder.basePrice;
return (basePrice > 1000);

//new
return anOrder.basePrice > 1000;
```

**动机**

- 表达式更容易被阅读、使用
- 妨碍了之后的代码操作

**步骤**

- 找到第一处使用该变量的地方，将其直接替换为表达式
- 测试
- 重复前两步




<br />

---

*To be continue*