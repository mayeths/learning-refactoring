# *Refactoring* 阅读笔记

[Refactoring: improving the design of existing code (2nd Edition)](https://refactoring.com/)

<p align="center">
  <img width="220" height="300" src="https://refactoring.com/refact2.jpg">
</p>

有条件者，推荐购买 [中文实体书籍](https://item.jd.com/12584498.html) 或电子版进行阅读。本书第二版使用了 `Javascript` 进行编写，而重构手法与语言无关（第一版为 `Java`）。此文档仅为个人笔记，不能包含所有内容（手法），且带有个人意见的附注和修改。本文档会有未记载的手法，如果需要的话，直接阅读原文会比较好。

本书中的「重构」一词表示一种特定的整理代码的方式，不会改变软件可观察的行为。这意味着，**严格按照重构手法修改的代码很少进入不可工作的状态**，即便重构没有完成，也（几乎）可以随时随地停下来。如果有人说他们的代码在重构时有一两天时间不可用，那可以确定他们做的不是重构，而是重写（p.46）。为了实现这样的目的，本书有很多时候是按照「复制原代码 → 修改至可以运作 → 替换原来使用的地方 → 清理旧代码」的工作流进行重构的。如果直接在原代码上修改，就违背了本书对于重构近乎严苛的定义。

重构不应对原行为产生影响，所以如果存在已被观测到的 bug ，重构结束后其依然要存在；如果没有还被观测到，也可以顺手修复它们。

阅读笔记的记录格式：
- 名称
- *简介（如果无法使用 200 字阐述，则阅读毫无意义）
- 代码展示（摘于 [官网目录](https://refactoring.com/catalog/) ）
- 动机（为什么要做这个重构 & 什么情况下不该做这个重构）
- 步骤（实现此重构遵循的流程）
- *笔记（其他思考）

使用 [GayHub](https://github.com/jawil/GayHub) 提高阅读体验

摘录结束感想：只有在坚持摘抄结束后，我才意识到，并不是所有的重构手法我都应该现在掌握，因为我的意识在摘抄的过程中已经得到了提高。为开发而进行重构才有意义，能辅助我们更好地完成任务是重构存在的意义，我们不需要为了重构而重构。它们之中的许多手法，都多多少少不经意间被我用过，而现在我也能大约感觉出它们来了。

*<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/80x15.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License.</a><br/>Welcome comments.*

---

## 目录
- [*Refactoring* 阅读笔记](#refactoring-阅读笔记)
  - [目录](#目录)
  - [书中部分术语的提前声明](#书中部分术语的提前声明)
    - [命令与查询](#命令与查询)
    - [记录，字段，属性](#记录字段属性)
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
    - [改变函数声明（Change Function Declaration）](#改变函数声明change-function-declaration)
    - [封装变量（Encapsulate Variable）](#封装变量encapsulate-variable)
    - [引入参数对象（Introduce Parameter Object）](#引入参数对象introduce-parameter-object)
    - [函数组合成类（Combine Functions into Class）](#函数组合成类combine-functions-into-class)
    - [函数组合成变换（Combine Functions into Transform）](#函数组合成变换combine-functions-into-transform)
    - [拆分阶段（Split Phase）](#拆分阶段split-phase)
  - [封装 p.161](#封装-p161)
    - [封装记录（Encapsulate Record）](#封装记录encapsulate-record)
    - [以对象取代基本类型（Replace Primitive with Object）](#以对象取代基本类型replace-primitive-with-object)
    - [以查询取代临时变量（Replace Temp with Query）](#以查询取代临时变量replace-temp-with-query)
    - [隐藏委托关系（Hide Delegate）](#隐藏委托关系hide-delegate)
    - [移除中间人（Remove Middle Man）](#移除中间人remove-middle-man)
  - [搬移特性 p.197](#搬移特性-p197)
    - [以函数调用取代内联代码（Replace Inline Code with Funtion Call）](#以函数调用取代内联代码replace-inline-code-with-funtion-call)
    - [拆分循环（Split Loop）](#拆分循环split-loop)
    - [以管道取代循环（Replace Loop with Pipline）](#以管道取代循环replace-loop-with-pipline)
  - [重新组织数据 p.239](#重新组织数据-p239)
    - [以查询取代派生变量（Replace Derived Variable with Query）](#以查询取代派生变量replace-derived-variable-with-query)
    - [将引用对象改为值对象（Change Reference to Value）](#将引用对象改为值对象change-reference-to-value)
    - [将值对象改为引用对象（Change Value to Reference）](#将值对象改为引用对象change-value-to-reference)
  - [简化条件逻辑 p.259](#简化条件逻辑-p259)
    - [分解条件表达式（Decompose Conditional）](#分解条件表达式decompose-conditional)
    - [合并条件表达式（Consolidate Conditional Expression）](#合并条件表达式consolidate-conditional-expression)
    - [以卫语句取代嵌套条件表达式（Replace Nested Conditional with Guard Clauses）](#以卫语句取代嵌套条件表达式replace-nested-conditional-with-guard-clauses)
    - [以多态取代条件表达式（Replace Conditional with Polymorphism）](#以多态取代条件表达式replace-conditional-with-polymorphism)
    - [引入特例（Introduce Special Case）](#引入特例introduce-special-case)
    - [引入断言（Introduce Assertion）](#引入断言introduce-assertion)
  - [重构 API p.305](#重构-api-p305)
    - [将查询函数与修改函数分离（Separate Query from Modifier）](#将查询函数与修改函数分离separate-query-from-modifier)
    - [移除标记函数（Remove Flag Argument）](#移除标记函数remove-flag-argument)
    - [保持对象完整（Preserve Whole Object）](#保持对象完整preserve-whole-object)
    - [以查询取代参数（Replace Parameter with Query）](#以查询取代参数replace-parameter-with-query)
    - [以查询取代参数（Replace Query with Parameter）](#以查询取代参数replace-query-with-parameter)
    - [移除设值函数（Remove Setting Method）](#移除设值函数remove-setting-method)
    - [以工厂函数取代构造函数（Replace Constructor with Factory Function）](#以工厂函数取代构造函数replace-constructor-with-factory-function)
    - [以命令取代函数（Replace Function with Command）](#以命令取代函数replace-function-with-command)
    - [以函数取代命令（Replace Command with Function）](#以函数取代命令replace-command-with-function)
  - [处理继承关系 p.349](#处理继承关系-p349)
    - [函数上移（Pull Up Method）](#函数上移pull-up-method)
    - [字段上移（Pull Up Field）](#字段上移pull-up-field)
    - [函数下移（Push Down Method）](#函数下移push-down-method)
    - [字段下移（Push Down Field）](#字段下移push-down-field)
    - [以子类取代类型码（Replace Type Code with Subclasses）](#以子类取代类型码replace-type-code-with-subclasses)
    - [移除子类（Remove Subclass）](#移除子类remove-subclass)
    - [提炼超类（Extract Superclass）](#提炼超类extract-superclass)
    - [以委托取代子类（Replace Subclass with Delegate）](#以委托取代子类replace-subclass-with-delegate)
    - [以委托取代超类（Replace Superclass with Delegate）](#以委托取代超类replace-superclass-with-delegate)

<br />


## 书中部分术语的提前声明

### 命令与查询

- 命令：带有副作用（与非局部变量进行交互，如 IO 、修改全局变量）的函数
- 查询：带有返回值的函数

一般来说，应该保持命令与查询的分离。查询保持纯函数的性质，不论多少次调用都不会产生影响。

### 记录，字段，属性

- 记录：完整数据的集合
- 字段：与对象或类关联的变量
- 属性：可以用 `get`、`set` 方法的数据

从前到后，它们一个比一个小。不需要纠结于它们的翻译导致意义不明确。

### 可变数据与不可变数据

- 可变数据：可以赋值和读取的数据
- 不可变数据：只可以读取的数据

可变数据意味着，一个实例的某些地方被暴露出来了，从而可以用赋值语句修改它。然而，不可变的数据是强大的代码防腐剂，它意为着你想要改变其内部时，只能通过创建新的数据来完成。不要对 `new` 抱有性能焦虑（见 [重构与性能](#重构与性能-p65)）。如果数据不可以修改，意味着你可以放心大胆地使用，而不用担心它会有其他逻辑导致内部发生改变。

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

什么时候应该开展重构？代码有坏味道的时候。尽管下面的坏味道描述很抽象，但不妨可以看一看。

- 神秘命名（Mysterious Name）
  - 命名是编程中最难的事之一（无头绪的时候，可以到 [codeif](https://unbug.github.io/codelf/) 去查翻译。有趣的是，网站的那句话就是从本书里摘抄的）。当出现不好的名字时，往往意为着更深层次的设计问题（如一个函数做了太多事情）。这时候可能会需要 `变量改名` 、`字段改名`。
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

重构很有价值，但只有重构还不行。要正确地使用重构，还得有一套牢固的测试集合，以帮助发现难以避免的疏漏。一个坚固的系统，应该使用一套合理的测试框架（如 [GTest](https://github.com/google/googletest)，[JUnit](https://github.com/junit-team/junit4) 等）构建一些自动化测试，让所有对系统的修改都能通过测试。

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
// Old
function printOwing(invoice) {
  printBanner();
  let outstanding  = calculateOutstanding();

  //print details
  console.log(`name: ${invoice.customer}`);
  console.log(`amount: ${outstanding}`);  
}

// New
function printOwing(invoice) {
  printBanner();
  let outstanding  = calculateOutstanding();
  printDetails(outstanding);

  function printDetails(outstanding) {
    console.log(`name: ${invoice.customer}`);
    console.log(`amount: ${outstanding}`);
  } // 利用了 Javascript 的函数提升
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
// Old
function getRating(driver) {
  return moreThanFiveLateDeliveries(driver) ? 2 : 1;
} // 代码展示中的名字不重要，重要的是它们的关系

function moreThanFiveLateDeliveries(driver) {
  return driver.numberOfLateDeliveries > 5;
}

// New
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
// Old
return order.quantity * order.itemPrice -
  Math.max(0, order.quantity - 500) * order.itemPrice * 0.05 +
  Math.min(order.quantity * order.itemPrice * 0.1, 100);

// New
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
// Old
let basePrice = anOrder.basePrice;
return (basePrice > 1000);

// New
return anOrder.basePrice > 1000;
```

**动机**

- 表达式更容易被阅读、使用
- 妨碍了之后的代码操作

**步骤**

- 找到第一处使用该变量的地方，将其直接替换为表达式
- 测试
- 重复前两步


### 改变函数声明（Change Function Declaration）

**简介**

改变函数声明有两种：一是改变函数名称，二是改变函数参数。第一种遵循文档最开始介绍的重构工作流就可以完成（复制原代码 → 修改至可以运作 → 替换原来使用的地方 → 清理旧代码）；第二种则会复杂一些，但它是下面 `引入参数对象` 的基础。

**代码展示**

```javascript
// Old
function inNewEngland(aCustomer) {
  return ["MA", "CT", "ME", "VT", "NH", "RI"].include(aCustomer.address.state);
} // 现在希望可以传入的是各个 customer 的 state 而不是他们自身

// Transition
function xxxinNewEngland(state) {
  return ["MA", "CT", "ME", "VT", "NH", "RI"].include(state);
} // 临时的新函数，这时候把所有调用点换成这个函数
function inNewEngland(aCustomer) {
  return ["MA", "CT", "ME", "VT", "NH", "RI"].include(aCustomer.address.state);
} // 待会再按照工作流删除它

// New
function inNewEngland(state) {
  return ["MA", "CT", "ME", "VT", "NH", "RI"].include(state);
} // 全局搜素将xxx前缀去掉并删除原函数
```

**动机**

- 函数名字无法和它当前的行为相匹配，或者有更好的名字
- 函数实现功能时与不必要的模块进行了耦合，例如代码展示中去掉 `Customer` 后可以增加函数的使用范围

**步骤**

- GGG

**笔记**

如果修改的是对外的 API，并且语言不支持声明「不推荐使用」（deprecated），那么你可能需要的是旧函数内转发而不是删除它。

如何选取适合的参数，这没有简单的规律可循。有时候，你可以通过耦合方便得到传入对象的其他属性，从而减少参数个数（`引入参数对象`）；而通过解耦，你可以得到函数更广的使用范围。对于这道难题，唯一正确的答案是「没有正确答案」。在设计阶段，确定职责分明、边界明晰的分工可以较好地解决这个问题。

<br />

### 封装变量（Encapsulate Variable）

**简介**

曾用名为 `封装字段`，是面向对象「对象的数据应该保持私有」的封装理念。一个变量的可访问范围变大时，其行为会愈发像全局变量，破坏系统可靠性。在 `Javascript` 等语言中，你可以通过 `get` `set` 关键词快速重构。否则，你应该自己暴露 `get` `set` 函数。如果可以转化为 `不可变数据` 的话，你可以更放心地使用它。

**代码展示**

```javascript
// Old
let defaultOwner = {firstName: "Martin", lastName: "Fowler"};

// New
let defaultOwnerData = {firstName: "Martin", lastName: "Fowler"};
export function defaultOwner()       {return defaultOwnerData;}
export function setDefaultOwner(arg) {defaultOwnerData = arg;}
```

**动机**

- 类的公开字段使用范围开始无法控制
- 因此而导致重构系统的难度变大

**步骤**

- 创建封装函数（`get` 和 `set`），在其中访问和更新变量值
- 逐一修改使用该变量的代码，为其选用合适的封装函数。每次修改后，进行一次测试（你可以通过全局搜索或修改原变量名利用静态检查找到所有使用该变量的地方）
- 限制变量对外不可见
- 如果变量的值是一个记录，则考虑使用 `封装记录`

**笔记**

更彻底的做法是，删除所有的设值函数（`set`），从而让它成为不可变的。想要修改它，那就复制一份重新构造。或者，让设值函数返回新的副本也可以（这时候，设值函数可能带有 `change` 字样）。取数据时返回一个副本的好处在于，这样可以防止因为源数据发生变化而造成意外事故。

也许很多时候没必要复制一份数据，但多复制一次对性能的影响通常可以忽略不计。如果不做复制，风险则是未来可能会陷入漫长而困难的调试排错过程。

### 引入参数对象（Introduce Parameter Object）

**简介**

一组总是结伴而行的数据叫「数据泥团」，组织它们是件有价值的事，这会让数据的关系变得更明晰。而这项重构真正的意义在于，它可以催生代码更深层次的改变。一旦有了新的数据结构，就可以重组程序的行为来使用它们，比如将有关的函数捕捉成新的类。新的数据结构会改变代码的图景，帮助你更好地理解问题。

**代码展示**

```javascript
// Old
function amountInvoiced(startDate, endDate) {...}
function amountReceived(startDate, endDate) {...}
function amountOverdue(startDate, endDate) {...}

// New
function amountInvoiced(aDateRange) {...}
function amountReceived(aDateRange) {...}
function amountOverdue(aDateRange) {...}
```

**动机**

- 有那么几个数据项总是结伴而行

**步骤**

- 如果还没有合适的数据结构，那就创建一个（通常，我们会使用类，并尽量确保它是 [值对象](https://martinfowler.com/bliki/ValueObject.html)）
- 测试
- 使用 `改变函数声明` 给原来的函数添加一个参数，类型是新的数据结构
- 测试
- 调整所有调用者，传入新数据结构的适当实例
- 用新数据结构中的元素逐一取代参数列表中与之对应的参数项，然后删除原来的函数
- 测试

**笔记**

重构不会导致原代码出现不可用的情况，因而你需要渐进式地改变代码。利用 `改变函数声明` 中添加一个参数的方法，可以在原代码不被破坏的情况下，将新的函数准备好之后一举转过去，最后再清理旧函数。


### 函数组合成类（Combine Functions into Class）

**简介**

类，是现代大多数语言的一等公民。它将一类函数和一类数据绑定在一起，从而修改核心数据时函数派生的数据也会自动与核心数据保持一致。「封装」概念使得面向对象编程在工业应用中一骑绝尘，我们在实现中也应该尽可能去靠拢。这个重构达到了这个目的。

**代码展示**

```javascript
// Old
function base(aReading) {...}
function taxableCharge(aReading) {...}
function calculateBaseCharge(aReading) {...}

// New
class Reading {
  base() {...}
  taxableCharge() {...}
  calculateBaseCharge() {...}
}
```

**动机**

- 一组函数与同一块数据形影不离

**步骤**

- 将多个函数共用的数据封装成一个对象
- 搬移使用该记录结构的每个函数到新的类
- 将散落的处理该数据记录的代码段提炼到信的类


### 函数组合成变换（Combine Functions into Transform）

**简介**

有些编程语言（例如纯粹的 `Haskell` 等函数式语言和 `Javascript` 等多范式语言），类不是一等公民，函数才是。[什么是函数式编程思维？](https://www.zhihu.com/question/28292740/answer/100284611)如果有机会的话，不妨可以学习一门函数式编程语言（`Haskell`），以变换作为解决问题的方法，让自己对问题有更深一层的认识。

**代码展示**

```javascript
// Old
function base(aReading) {...}
function taxableCharge(aReading) {...}

// New
function enrichReading(argReading) {
  const aReading = _.cloneDeep(argReading);
  aReading.baseCharge = base(aReading);
  aReading.taxableCharge = taxableCharge(aReading);
  return aReading;
}
```

**动机**

- 当你想试一试用变换解决问题

**步骤**

- 创建一个变换函数，输入参数是需要变换的记录，并直接返回该记录的值
- 挑选一块逻辑，将其主体移入变换函数中，把结果作为字段添加到输出记录中。修改调用点使用此字段
- 测试
- 重复上述步骤

**笔记**

如果你打算在类不是一等公民的语言中用上一个手法，那么可以用 [函数作为对象](https://martinfowler.com/bliki/FunctionAsObject.html) 的形式来实现；而如果你打算映射到底，那就用本手法。

使用哪种手法需要根据你的已有代码来选择。一般来说，上一个 `函数组合成类` 会更符合我们的需求。


### 拆分阶段（Split Phase）

**简介**

这个几乎是最基本的一个重构手法了：一段代码在处理两件不同的事时，把它们拆分成各自独立的模块。这样到了需要修改的时候，你就可以单独处理每个主题，而不必在脑子里考虑两个不同的主题。

**代码展示**

```javascript
// Old
const orderData = orderString.split(/\s+/);
const productPrice = priceList[orderData[0].split("-")[1]];
const orderPrice = parseInt(orderData[1]) * productPrice;

// New
const orderRecord = parseOrder(order);
const orderPrice = price(orderRecord, priceList);

function parseOrder(aString) {
  const values =  aString.split(/\s+/);
  return ({
    productID: values[0].split("-")[1],
    quantity: parseInt(values[1]),
  });
}
function price(order, priceList) {
  return order.quantity * priceList[order.productID];
}
```

**动机**

- 一段代码在处理两件不同的事情

**步骤**

- 将第二阶段的函数提炼成独立的函数
- 测试
- 引入一个中转数据结构，将其作为参数添加到提炼的新函数参数列表中
- 测试
- 逐一提炼出第二阶段函数的每个参数，如果某一参数在第一阶段被用到，则将其移入中转数据结构中
- 测试
- 对第一阶段代码使用 `提炼函数`，让提炼出来的函数返回中转数据结构

**笔记**

如果一个函数做了太多的事情，那就拆分它——我们一直是这么说的。

<br />


## 封装 p.161

### 封装记录（Encapsulate Record）

**简介**

记录型结构是多数编程语言提供的常见特性（`javascript` 的对象、`python` 的字典），但记录只会提供源数据，而无法将派生数据的制造与它们关联起来。用类可以很好地解决这一问题。

**代码展示**

```javascript 
// Old
organization = {name: "Acme Gooseberries", country: "GB"};

// New
class Organization {
  constructor(data) {
    this._name = data.name;
    this._country = data.country;
  }
  get name()    {return this._name;}
  set name(arg) {this._name = arg;}
  get country()    {return this._country;}
  set country(arg) {this._country = arg;}
}
```

**动机**

- 数据经常有派生计算
- 用户不想知道结构的细节，比如一个值是派生的还是原生的

**步骤**

- 对持有记录的变量使用 `封装变量`，将其装到一个具有易于搜索的函数名的函数中
- 创建一个类，将记录包装起来。并将记录变量的值替换为该类的一个实例。然后在类上定义一个访问函数，用于返回原始的记录。修改封装变量的函数，令其使用这个函数
- 测试
- 新建一个函数，让它返回该类的对象而不是原始的记录
- 对于该记录的每处调用点，将原先返回记录的函数调用替换为那个返回实例对象的函数调用。使用对象上的访问函数来获取数据的字段。
- 移除类对原始对象的访问函数，那个容易搜索的返回原始对象的函数也要一并删除。
- 测试

**笔记**

尽量使用类来封装数据而不是利用记录——这在 `Javascript` 的大型程序中可以让一个被到处传递的数据更加易读。

### 以对象取代基本类型（Replace Primitive with Object）

**简介**

开发初期，我们通常会使用基本数据类型（`Javascript` 的 `{}` 对象）来表示简单情况。然而随着开发继续，你会不断添加新的东西。是时候将它封装为一个类了。

**代码展示**

```javascript
// Old
orders.filter(o => "high" === o.priority
                || "rush" === o.priority);

// New
orders.filter(o => o.priority.higherThan(new Priority("normal")))
```

**动机**

- 某个数据类型的操作不仅仅局限于打印
- 数据结构不再只是承担简单的任务

**步骤**

- 如果变量还未封装起来，使用 `封装变量` 封装它
- 为这个数据值创建一个简单的类
- 执行静态检查
- 修改第一步得到的设值函数，令其创建一个新类的对象并将其存入字段
- 修改取值函数，令其调用新类的取值函数，并返回结果
- 测试

**笔记**

这些小小的封装开始可能价值甚微，但只要悉心照料，它们很快便能成长为有用的工具。创建新的类无需太大的工作量，但它们往往对代码库有深远的影响。

### 以查询取代临时变量（Replace Temp with Query）

**简介**

如果正在分解一个冗长的函数，那么将变量抽取到函数里能使函数的分解过程更加简单。改用函数还能避免未来在多个函数里重复编写计算逻辑。

**代码展示**

```javascript
// Old
const basePrice = this._quantity * this._itemPrice;
if (basePrice > 1000)
  return basePrice * 0.95;
else
  return basePrice * 0.98;

// New
get basePrice() {this._quantity * this._itemPrice;}

...

if (this.basePrice > 1000)
  return this.basePrice * 0.95;
else
  return this.basePrice * 0.98;
```

**动机**

- 分解类中冗长的函数

**步骤**

- 检查变量在使用前是否已经计算完毕，检查它的赋值代码是否每次都能得到相同的值
- 如果变量不是只读的，但是可以改造成只读变量，那就先改造它
- 测试
- 将为变量赋值的代码提炼成函数
- 测试

**笔记**

这项重构手法在类里施展的效果最好，因为类为待提炼函数提供了一个和原来一样的上下文。

### 隐藏委托关系（Hide Delegate）

**简介**

尽可能少地暴露对象的内部特征，即尽可能少地暴露对象的字段。

**代码展示**

```javascript
// Old
manager = aPerson.department.manager;

// New
manager = aPerson.manager;

class Person {
  get manager() {return this.department.manager;}
```

**动机**

- 某些客户端需要先通过对象的字段得到另一个对象，然后再去调用自己想要的函数

**步骤**

- 对于每个委托关系中的函数，在服务对象端建立一个简单的委托函数
- 调整客户端，令它只调用服务对象提供的函数，每次调整后进行测试
- 如果将来不再有任何客户端需要调用 Delegate（受托类），便可移除服务对象中的相关访问函数
- 测试

**笔记**

不要暴露对象的工作原理——我只想通过你来得到相关的信息，不想去知晓你的实现


### 移除中间人（Remove Middle Man）

**简介**

上一个手法也是有代价的——每当使用受托类的新特性时，你就必须添加一个简单的委托函数。随着受委托的特性越来越多，太多的转发函数会让人烦躁。很难说隐藏到什么程度是合适的，因为你会在系统运行的过程中不断调整这个度。重构的意义在于，你永远不用说「对不起」——只要把出问题的地方修补好就行了。

**代码展示**

```javascript
// Old
manager = aPerson.manager;

class Person {
  get manager() {return this.department.manager;}

// New
manager = aPerson.department.manager;
```

**动机**

- 受委托（转发）的特性越来越多

**步骤**

- 为受委托的对象创建一个取指函数
- 对于每个委托函数，让其客户端转为连续的访问函数调用
- 测试
- 删除掉没用了的委托方法


<br />


## 搬移特性 p.197

### 以函数调用取代内联代码（Replace Inline Code with Funtion Call）

**简介**

对于现代大多数语言（哪怕是 `C++`），官方库已经提供了很好的封装函数——尤其是对于数组的操作来说。能否完全发掘官方库的潜力只取决于你对它们的理解程度。

**代码展示**

```javascript
// Old
let appliesToMass = false;
for(const s of states) {
  if (s === "MA") appliesToMass = true;
}

// New
appliesToMass = states.includes("MA");
```

**动机**

- 代码在做重复的事情——自己定义一些已经存在了的操作

**步骤**

- 将内联代码替换为一个既有函数的调用
- 测试

**笔记**

`Modern C++`，其友好程度已经赶上 `Python`、`Javascript` 等语言了。现在几乎没有哪种语言不提供对数据结构最常用的操作，你还有什么理由不尝试一下？


### 拆分循环（Split Loop）

**简介**

谁没见过一些循环干好几种事情？不为别的，就为了能只循环一次。然而，如果你想改动其中一件事，你就得了解其他的事。拆分循环后，你很可能会想立即做一个 `提炼函数` 的重构。

**代码展示**

```javascript
// Old
let averageAge = 0;
let totalSalary = 0;
for (const p of people) {
  averageAge += p.age;
  totalSalary += p.salary;
}
averageAge = averageAge / people.length;

// New
let totalSalary = 0;
for (const p of people) {
  totalSalary += p.salary;
}

let averageAge = 0;
for (const p of people) {
  averageAge += p.age;
}
averageAge = averageAge / people.length;
```

**动机**

- 一个循环做了太多的事情

**步骤**

- 复制一遍循环代码
- 识别并移除循环中的重复代码，每个循环只做一件事
- 测试并考虑做 `提炼函数` 的重构

**笔记**

这项重构可能会让许多程序员坐立难安——循环导致的性能下降不会很明显吗？然而，我们建议也与之前说的一致——先进行重构，然后再做性能优化。计算机没你想象的那么不堪。拆分出来的循环也让一些更强大的优化手段成为了可能。

然而，很多时候我们很难抵挡 `以管道取代循环` 手法的诱惑。

<br />

### 以管道取代循环（Replace Loop with Pipline）

**简介**

> 我刚入门的时候也有人告诉我，迭代一组集合时得用循环。不过时代在发展，越来越多的编程语言提供了更好的语言结构来处理迭代过程，这种结构就叫集合管道。这类管道最常见的非 `map` 和 `filter` 莫属。——p.231

**代码展示**

```javascript
// Old
const names = [];
for (const i of input) {
  if (i.job === "programmer")
    names.push(i.name);
}

// New
const names = input
  .filter(i => i.job === "programmer")
  .map(i => i.name)
;
```

**动机**

- 希望可以提高代码的可读性或其他

**步骤**

- 创建一个新变量用于存放参与循环过程的集合
- 从循环顶部开始，将循环的每一块行为一次搬移出来，在上一步建立的变量上用一种管道运算替代之。每次修改后测试
- 将循环整个删除

**笔记**

如果你有研读过之前谈到函数式语言给的的链接时，你会发现这和函数式最基本的思想一致——最终处理结果应该是映射出来的（即使函数式没那么简单）。


<br />


## 重新组织数据 p.239

### 以查询取代派生变量（Replace Derived Variable with Query）

**简介**

可变数据是软件最大的错误源头之一，对数据的修改常常导致代码的各个部分以丑陋的形式耦合在一起。有些变量其实很容易被随时算出来，如果能消除他们，那么也算朝着消除可变性的方向迈了一大步。

**代码展示**

```javascript
// Old
get discountedTotal() {return this._discountedTotal;}
set discount(aNumber) {
  const old = this._discount;
  this._discount = aNumber;
  this._discountedTotal += old - aNumber; 
}

// New
get discountedTotal() {return this._baseTotal - this._discount;}
set discount(aNumber) {this._discount = aNumber;}
```

**动机**

- 到处都在修改可变数据时

**步骤**

为了说明情况，以一个人当天的消费情况总数 `costNow` 来说明：

- 识别所有对该可变变量做更新的地方
- 新建一个函数，用于计算该变量的值（通过遍历当天历史记录算出）
- `引入断言` 断言该变量和计算函数始终给出同样的值
- 测试
- 修改读取该变量的代码，令其调用新的函数
- 测试
- 移除死代码

**笔记**

可以随时算出来，那就不要记录了。因为可变数据可能会无法保持同步，实在是「ugly」。

### 将引用对象改为值对象（Change Reference to Value）

**简介**

值对象就是典型的不可变对象。想要更新它？请重新构造一个。不可变的数据处理起来总是令人愉悦。

**代码展示**

```javascript
// Old
class Product {
  applyDiscount(arg) {this._price.amount -= arg;}

// New
class Product {
  applyDiscount(arg) {
    this._price = new Money(this._price.amount - arg, this._price.currency);
  }
```

**动机**

- 拥抱不可变，拥抱安心

**步骤**

- 检查重构目标是否为不可变对象，或者是否可修改为不可变对象
- 用 `移除设置函数` 逐一去掉所有的设值函数
- 提供一个基于值的相等性判断函数，在其中使用值对象的字段

**笔记**

基本数据单元设为值对象是一件有益的事

### 将值对象改为引用对象（Change Value to Reference）

**简介**

当然，不是所有对象都应该是值对象。当你想在几个对象之间共享一个对象（通常，单例模式就是这样的情况），以便它们可以共享那个对象的修改，那么这个对象就应该是引用对象。或者，你想维护一系列顾客数据（这时候的顾客类意在一种身份而不是实例），也应该用引用对象。

**代码展示**

```javascript
// Old
let customer = new Customer(customerData);

// New
let customer = customerRepository.get(customerData.id);
```

**动机**

- 几个对象需要共享一个对象

**步骤**

- 为相关对象创建一个仓库（不要只是一个数组，它可以成为单例类）
- 确保构造函数有办法找到关联对象中的正确实例
  - 例如，用 `uuid` 来确保仓库内只有一份正确实例
- 修改宿主对象的构造函数，令其从仓库中获取关联对象。每次修改后执行测试

**笔记**

你应该在不断的重构中对问题架构有一个越来越清晰的认识，从而能辨别哪些是值对象，哪些是引用对象。一般来说，基本的数据结构（`Phone`）为值对象，代表实际身份的数据结构（`Customer`）为引用对象比较合理。

<br />


## 简化条件逻辑 p.259

### 分解条件表达式（Decompose Conditional）

**简介**

程序的大部分威力来自于条件逻辑，但很不幸，程序的复杂也大多来自于条件逻辑。借助重构将条件逻辑变得更容易理解，有助于让代码变得健康。

**代码展示**

```javascript
// Old
if (!aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd))
  charge = quantity * plan.summerRate;
else
  charge = quantity * plan.regularRate + plan.regularServiceCharge;

// New
if (summer())
  charge = summerCharge();
else
  charge = regularCharge();
```

**动机**

- 大型函数本身已经导致代码可读性下降，而条件逻辑更会导致代码难以理解

**步骤**

- 为判断语句和每个条件分支使用 `提炼函数` 的手法

**笔记**

你终于要开始重构条件逻辑（`if else`、`switch case`）了吗？太棒了！这是重构里最有益的部分！

### 合并条件表达式（Consolidate Conditional Expression）

**简介**

有时候会有多个条件表达式有相同的结果，那么合并它们可以让条件检查的用意更加明晰，并且可以为 `提炼函数` 做准备。

**代码展示**

```javascript
// Old
if (anEmployee.seniority < 2) return 0;
if (anEmployee.monthsDisabled > 12) return 0;
if (anEmployee.isPartTime) return 0;

// New
if (isNotEligableForDisability()) return 0;

function isNotEligableForDisability() {
  return ((anEmployee.seniority < 2)
          || (anEmployee.monthsDisabled > 12)
          || (anEmployee.isPartTime));
}
```

**动机**

- 有多个条件表达式有相同的结果

**步骤**

- 确定这些表达式没有副作用。否则，先用 `将查询函数与修改函数分离`
- 用适当的与或非运算符来合并表达式
- 测试
- 重复上面的过程直到所有相关的表达式都合并到了一起
- 考虑对合并之后的条件表达式使用 `提炼函数` 

**笔记**

条件合并的理由同时也指出了不要合并的理由：如果这些检查彼此独立，那么的确不应该视为同一次检查。

### 以卫语句取代嵌套条件表达式（Replace Nested Conditional with Guard Clauses）

**简介**

有 [这么一个问题](https://www.zhihu.com/question/344856665) 困扰了许多程序员很长时间，他们也各自提出了自己的解决方案。那么，下面也会提供几种解决方案，根据你的需要进行选择。有两种风格的条件分支语句：两个条件都属于正常行为或者一个正常，另一个则是异常。那个罕见而特殊的单独检查就叫「卫语句」。这项重构的精髓就是，给某一分支特别的重视。

**代码展示**

```javascript
// Old
function getPayAmount() {
  let result;
  if (isDead)
    result = deadAmount();
  else {
    if (isSeparated)
      result = separatedAmount();
    else {
      if (isRetired)
        result = retiredAmount();
      else
        result = normalPayAmount();
    }
  }
  return result;
}

// New
function getPayAmount() {
  if (isDead) return deadAmount();
  if (isSeparated) return separatedAmount();
  if (isRetired) return retiredAmount();
  return normalPayAmount();
}
```

**动机**

- 函数里有卫语句

**步骤**

- 选中最外层需要被替换的条件逻辑，将其替换为卫语句
- 测试
- 重复上述步骤
- 如果所有的卫语句都引发同样的结果，则使用 `合并条件表达式` 合并之

**笔记**

卫语句应该告诉读者：「这种情况不是本函数核心逻辑所关心的，如果它真的发生了，那就做一些必要的工作然后退出。」

### 以多态取代条件表达式（Replace Conditional with Polymorphism）

**简介**

大多数条件逻辑其实是为了单一直观的逻辑而存在的。而如果某个类里有好几个条件逻辑，它们根据某个标识参数进行区分自己的表现行为，那么这种就可以根据标识参数进行多态处理。

**代码展示**

```javascript
// Old
switch (bird.type) {
  case 'EuropeanSwallow':
    return "average";
  case 'AfricanSwallow':
    return (bird.numberOfCoconuts > 2) ? "tired" : "average";
  case 'NorwegianBlueParrot':
    return (bird.voltage > 100) ? "scorched" : "beautiful";
  default:
    return "unknown";

// New
class EuropeanSwallow {
  get plumage() {
    return "average";
  }
class AfricanSwallow {
  get plumage() {
     return (this.numberOfCoconuts > 2) ? "tired" : "average";
  }
class NorwegianBlueParrot {
  get plumage() {
     return (this.voltage > 100) ? "scorched" : "beautiful";
  }
```

**动机**

- 一个基础逻辑之上，又存在一些变体

**步骤**

- 如果当前的类不具备多态性为，就用 `工厂函数` 创建之
- 在调用方使用工厂函数获得对象实例
- 将带有条件逻辑的函数移到超类中
- 任选一个子类在其中创建一个函数，根据对应的条件逻辑覆写超类中的那个函数并进行调整
- 重复上述步骤
- 在超类中保留默认情况的逻辑。如果超类应该是抽象的，就把该函数声明为 `abstract` 或者抛出异常。

**笔记**

经常发生这么一件事情：某一个类里有一个叫 `type` 的 enum 型字段，用于区分其代表的实体身份。当这种情况发生，就应该考虑是否可以将其使用多态处理。

### 引入特例（Introduce Special Case）

**简介**

经常会出现这样一种情况：一个数据结构的使用者总是检查这个数据结构的某一特殊值，并且对这个特殊值做相同的处理。这种情况就是「特例」。

**代码展示**

```javascript
// Old
if (aCustomer === "unknown") customerName = "occupant";

// New
class UnknownCustomer {
    get name() {return "occupant";}
```

**动机**

- 一种数据结构可能会有特殊的情况，并且这种特殊情况的存在是有意义的

**步骤**

- 给重构目标添加一个检查特例的属性，令其返回 `false`
- 创建一个特例对象，其中只有检查特例的属性，返回 `true`
- 对「与特例值做比对」的代码运用 `提炼函数`，确保所有客户端都使用这个新函数，而不再直接与特例值做比对
- 将新的特例对象引入代码中，可以从函数调用中返回，也可以在变换函数中生成
- 修改特例比对函数的主体，在其中直接使用检查特例的属性
- 测试
- 使用 `函数组合成类` 或者 `函数组合成变换`，把通用的特例处理逻辑都搬移到新建的特例对象中
- 对特例比对函数使用内联函数，将其内联到仍然需要的地方

**笔记**

如果担心抽象类派生的子类可能无法覆盖特例情况，有两种解决方式：一是使抽象类不可实例化，二是使用一个特例类来容纳特殊情况。

### 引入断言（Introduce Assertion）

**简介**

常常会有这样的代码：当某个条件为真时，这段代码才能正常工作。例如，平方根函数只能在正数时运算。这样的假设通常不会在代码中明确表现出来，有可能是通过注释来表现的。现在可以使用断言来清晰表达程序员犯了错误，比如在调用该函数前没有对输入的有效性进行判断。

**代码展示**

```javascript
// Old
if (this.discountRate)
  base = base - (this.discountRate * base);

// New
assert(this.discountRate >= 0);
if (this.discountRate)
  base = base - (this.discountRate * base);
```

**动机**

- 用断言来表明程序在这里对输入或其他条件进行了哪些假设，如果无法匹配这些假设，那么程序就有可能出现 undefined 的行为（例如崩溃）

**步骤**

- 如果发现代码要求某个条件为真，就加入一个断言说明这种情况

**笔记**

这对于 C++ 传递对象指针时很有用：如果对象指针为 nullptr，则抛出一个异常以便于程序员在这里直接展开调查。

断言应该总是为真（意味着它只应该出现在永远为真的地方），正常运行时它不是你的工具——如果它在运行时发生失败，表明程序员犯了错误（而不是用户）。


## 重构 API p.305

### 将查询函数与修改函数分离（Separate Query from Modifier）

**简介**

一个只提供一个值而没有任何看得到的副作用的函数是很有价值的东西，因为我们可以随意调用这个函数。一个好规则是，任何有返回值的函数都不应该有看得到的副作用——例如修改某个外界的计量值。这就是命令与查询分离原则。试着将命令和查询分离，在外界分别调用它们。

**代码展示**

```javascript
// Old
function getTotalOutstandingAndSendBill() {
  const result = customer.invoices.reduce((total, each) => each.amount + total, 0);
  sendBill();
  return result;
}

// New
function totalOutstanding() {
  return customer.invoices.reduce((total, each) => each.amount + total, 0);  
}
function sendBill() {
  emailGateway.send(formatBill(customer));
}
```

**动机**

- 发现一个既有返回值又有看得到的副作用的函数

**步骤**

- 复制整个函数，将其作为一个查询来命名
- 从新建的查询函数中去掉所有造成副作用的语句
- 执行静态检查
- 查找所有使用原函数的地方，如果调用出用到了该函数的返回值，就将其改为调用新建的查询函数，并在下面马上再调用一次原函数。每次修改之后都要测试
- 从原函数中去掉返回值和重复代码
- 测试

**笔记**

并非绝对遵守这个原则，而是尽量遵守它。这样做可以回报很好的效果，而如果原来的函数太难于改动就另想办法。

### 移除标记函数（Remove Flag Argument）

**简介**

「标记参数」用来让调用者指示目标函数应该执行哪一部分逻辑，通常它会是一个 `bool` 型变量。而如果传入的参数是程序中流动的数据流，那么这样的就不算是标记参数。移除标记参数可以让代码更简洁，尤其是一个使用了太多标记参数的函数，会不禁让人觉得它是不是做了太多事情。

**代码展示**

```javascript
// Old
function setDimension(name, value) {
  if (name === "height") {
    this._height = value;
    return;
  }
  if (name === "width") {
    this._width = value;
    return;
  }
}

// New
function setHeight(value) {this._height = value;}
function setWidth (value) {this._width = value;}
```

**动机**

- 函数的功能可以通过分为多个函数或者使用多态性来表现
- 函数依赖于 `bool` 值参数来决定执行的逻辑

**步骤**

- 针对标记参数的每一种可能值，新建一个明确参数
- 对于「用字面量值作为参数」的函数调用者，将其改为调用新建的明确参数

**笔记**

这种多个逻辑混用一个执行函数的方式实在是令人不悦，即使是在外面再套一层函数用于分支管理都比这样好得多。

### 保持对象完整（Preserve Whole Object）

**简介**

传递整个对象可以更好地应对变化，而有时候你可能不想采用这种手法——如果这些数据用在不同对象的时候，你应该将用到的数据封装一个新的字段。

**代码展示**

```javascript
// Old
const low = aRoom.daysTempRange.low;
const high = aRoom.daysTempRange.high;
if (aPlan.withinRange(low, high))

// New
if (aPlan.withinRange(aRoom.daysTempRange))
```

**动机**

- 一个对象的值被拆分了出来并传给另一个函数

**步骤**

- 创建一个空函数，给它以期望中的参数列表（即传入完整对象作为参数）
- 在新的函数体内调用旧函数，并把新的参数（即完整对象）映射到旧的参数列表
- 执行静态检查
- 逐一修改旧函数的调用者令其使用新函数，每次修改后测试
- 使用内联函数把旧函数内联到新函数体内
- 给新函数改名为旧函数的名字

**笔记**

传递整个对象有助于应对函数体内对新数据的需求变化。

### 以查询取代参数（Replace Parameter with Query）

**简介**

如果调用函数传入了一个值，这个值又函数自己获得也是同样容易，那么就没有必要传入。

**代码展示**

```javascript
// Old
availableVacation(anEmployee, anEmployee.grade);

function availableVacation(anEmployee, grade) {
  // calculate vacation...

// New
availableVacation(anEmployee);

function availableVacation(anEmployee) {
  const grade = anEmployee.grade;
  // calculate vacation...
```

**动机**

- 函数可以计算或者简单查询就可以获取正确的值
- 简化调用方行为

**步骤**

- 如果有必要，使用提炼函数将函数的计算过程提炼到一个独立的函数中
- 将函数体内引用该函数的地方改为调用新建的函数，每次修改后测试
- 全部替换完成后，使用「改变函数声明」来将该参数去掉

**笔记**

如果处理的函数具有「引用透明性」，即不论什么时候传入相同的值其行为总是一致的，这样的函数就拥有了既容易理解又容易调试的优良品质。如果是这样，那么你就不必要让其去查询一个全局变量。

### 以查询取代参数（Replace Query with Parameter）

**简介**

如果把所有依赖关系都变成参数，会导致参数列表冗长重复；而如果作用域之间的共享太多，会导致函数的依赖过度。如果你不善于处理微妙的平衡，熟练使用重构会让你能够可靠地改变决定。

**代码展示**

```javascript
// Old
targetTemperature(aPlan)

function targetTemperature(aPlan) {
  currentTemperature = thermostat.currentTemperature;
  ...
}

// New
targetTemperature(aPlan, thermostat.currentTemperature)

function targetTemperature(aPlan, currentTemperature) {
  ...
}
```

**动机**

- 函数被迫使着访问某个程序函元素
- 函数距离「引用透明性」非常近

**步骤**

- 对执行查询操作的代码使用「提炼变量」，将其从函数体中分离出来
- 现在函数体代码已经不再执行查询操作（是前面提炼的变量），对这部分代码使用「提炼函数」
- 使用内联变量，消除刚才提炼出来的变量
- 对原来的函数使用「内联函数」
- 将新函数改回原来的函数名字

**笔记**

构建大型项目时，程序中责任分配问题永远没有一个一劳永逸的解决方案。这个重构与其反重构是构建大型项目时理应能够熟悉使用的手法。

### 移除设值函数（Remove Setting Method）

**简介**

这是我最喜欢的重构手法——因为这能让对象成为值对象，从而拥有不可变性。移除设置函数，还可以清晰地表达「构造之后不应该再更新这个字段」

**代码展示**

```javascript
// Old
class Person {
  get name() {...}
  set name(aString) {...}

// New
class Person {
  get name() {...}
```

**动机**

- 几乎只有构造函数会调用当前设值函数
- 对象是在客户端由创建脚本构造出来的（调用构造函数后调用各设值函数），然后字段不再修改

**步骤**

- 如果构造函数无法得到想要设入字段的值，就使用「改变函数声明」将这个值以参数的形式传入
- 移除所有在构造函数之外对设值函数的调用，改为使用新的构造函数
  - 如果不能把调用设值函数改为创建一个新对象，则放弃这项重构
- 使用内联函数消去设值函数。如果可以的话，将字段声明为不可变
- 测试

**笔记**

结合本书对不可变数据的推崇和 Qt 推崇中把对象看作是一种「身份」，在用户侧的应用中可以把基础而细小的类作为不可变对象，而 UI类、单例类等则应该看做是「身份」。

### 以工厂函数取代构造函数（Replace Constructor with Factory Function）

**简介**

构造函数具有一定的局限性，比如很多时候需要使用 `new` 特殊关键字来生成，也无法使用比默认函数更清晰的名字。

**代码展示**

```javascript
// Old
leadEngineer = new Employee(document.leadEngineer, 'E');

// New
leadEngineer = createEngineer(document.leadEngineer);
```

**动机**

- 需要根据环境或者参数信息返回子类或代理对象

**步骤**

- 新建一个工厂函数，让他调用现有的构造函数
- 将调用构造函数的代码改为调用工厂函数
- 每修改一处，就执行测试
- 尽量修改构造函数的可见范围

**笔记**

设计模式中的工厂模式是很常用的一种设计模式，改用工厂函数可以更好地规范调用代码。

### 以命令取代函数（Replace Function with Command）

**简介**

将函数封装成自己一个对象，叫做「命令对象」，或简称「命令」。这样的对象大多数只服务于单一函数，获得对该函数的请求，执行该函数，就是这种对象存在的意义。通过在命令对象上添加附加操作（如撤回），可以获得更大的控制灵活性和更强的表达能力，

**代码展示**

```javascript
// Old
function score(candidate, medicalExam, scoringGuide) {
  let result = 0;
  let healthLevel = 0;
  ...
}

// New
class Scorer {
  constructor(candidate, medicalExam, scoringGuide) {
    this._candidate = candidate;
    this._medicalExam = medicalExam;
    this._scoringGuide = scoringGuide;
  }

  execute() {
    this._result = 0;
    this._healthLevel = 0;
    ...
  }
}
```

**动机**

- 编程语言不支持函数作为一等公民
- 需要更丰富的生命周期管理能力

**步骤**

- 为想要包装的函数创建一个新的类，根据函数的名字命名
- 使用 `搬移函数` 把函数搬移到空的类里
  - 保持原来的函数作为转发函数，至少保留到重构结束后再删除
- 可以考虑为每个原函数参数创建一个字段，并在构造函数内添加相应的参数

**笔记**

使用命令对象有很多好理由，但90%的情况下我们只需要一个普通的执行函数，因而不需要滥用命令函数。

### 以函数取代命令（Replace Command with Function）

**简介**

这是上一个手法的反重构。虽然命令对象提供了强大的状态管理能力，但这种能力也是有代价的。如果函数不是太复杂，就应该将其变回普通的函数。

**代码展示**

```javascript
// Old
class ChargeCalculator {
  constructor (customer, usage){
    this._customer = customer;
    this._usage = usage;
  }
  execute() {
    return this._customer.rate * this._usage;
  }
}

// New
function charge(customer, usage) {
  return customer.rate * usage;
}
```

**动机**

- 命令对象引入了不实用的复杂性

**步骤**

- 运用 `提炼函数` 把「创建并执行命令对象」的代码单独提炼到一个函数中
  - 这会创建一个新的函数，由它来承载命令对象的功能
- 对命令对象执行时使用到的函数，逐个使用「内联函数」
- 使用 `改变函数声明` 把构造函数的参数转移到执行函数
- 对于所有的字段，在执行函数中找到引用它们的地方，并改为使用参数
  - 每次修改后都要测试
- 把「调用构造函数」和「调用执行函数」两步都内联到调用方（也就是替换命令对象的那个函数）
- 测试
- 用 `移除死代码` 把命令类消去

**笔记**

我也就是在 Qt 中用 `QUndoCommand` 那种需要引入撤回操作的时候才用到命令对象，函数已经够用了其实。


## 处理继承关系 p.349

### 函数上移（Pull Up Method）

**简介**

避免重复代码是很重要的，因为你很可能会经历「修改了其中一个却未能修改另一个」的风险。

**代码展示**

```javascript
// Old
class Employee {...}
class Salesman extends Employee {
  get name() {...}
}
class Engineer extends Employee {
  get name() {...}
}

// New
class Employee {
  get name() {...}
}
class Salesman extends Employee {...}
class Engineer extends Employee {...}

```

**动机**

- 子类重复实现了相同的函数

**步骤**

- 检查待提升函数确定他们是完全一致的
- 检查函数体内所有的函数调用和字段都可以在超类中使用到
  - 否则，先把它们提升到超类中。如果无法实现，就放弃重构
- 在超类中新建一个函数，将其中一个待提升的代码复制到其中
- 执行检查
- 移除一个待提升的子类函数
- 执行测试
- 重复移除所有的子类函数并时刻执行测试

**笔记**

如果两个函数工作流程大体相似，但实现细节存在差异，那么可以考虑先 `构造模板函数` 构造出相同的函数，再提升他们。

### 字段上移（Pull Up Field）

**代码展示**

```java
// Old
class Employee {...} // Java
class Salesman extends Employee {
  private String name;
}
class Engineer extends Employee {
  private String name;
}

// New
class Employee {
  protected String name;
}
class Salesman extends Employee {...}
class Engineer extends Employee {...}

```

**动机**

- 去除重复的数据声明

**步骤**

- 针对待提升之字段，检查它们的所有使用点，确认它们以相同的方式被使用
- 如果这些字段的名字不同，先使用 `变量改名` 为它们取个相同的名字
- 在超类中新建一个字段，并根据子类字段的可见性合理设置其可见性
- 移除子类中的字段
- 测试

### 函数下移（Push Down Method）

**代码展示**

```javascript
// Old
class Employee {
  get quota {...}
}
class Engineer extends Employee {...}
class Salesman extends Employee {...}

// New
class Employee {...}
class Engineer extends Employee {...}
class Salesman extends Employee {
  get quota {...}  
}
```

**动机**

- 超类中的某些函数只与少数几个子类有关

**步骤**

- 将超类中的函数本体复制到每一个需要此函数的子类中
- 删除超类中的函数

### 字段下移（Push Down Field）

**代码展示**

```javascript
// Old
class Employee {   // Java
  private String quota;
}
class Engineer extends Employee {...}
class Salesman extends Employee {...}

// New
class Employee {...}
class Engineer extends Employee {...}
class Salesman extends Employee {
  protected String quota;
}
```

**动机**

- 超类中的某些字段只与少数几个子类有关

**步骤**

- 在所有需要该字段的子类内声明该字段
- 将该字段从超类中删除
- 测试

### 以子类取代类型码（Replace Type Code with Subclasses）

**简介**

软件通常需要表现「相似但又不同的东西」，比如员工分类（工程师，经理，销售），订单按优先级分类（加急、常规）。表现关系的一种常见形式是类型码字段——比如枚举、符号、数字等。但引入子类有两个诱人之处：你可以用多态来处理条件逻辑；有些字段和函数只对特定的类型码取值才有意义（销售目标对销售员才有意义，可以把这个字段下移到合适的类中去）。

**代码展示**

```javascript
// Old
function createEmployee(name, type) {
  return new Employee(name, type);
}

// New
function createEmployee(name, type) {
  switch (type) {
    case "engineer": return new Engineer(name);
    case "salesman": return new Salesman(name);
    case "manager":  return new Manager (name);
  }
}
```

**动机**

- 有几个函数根据条件码的取值采取不同的行为
- 有些函数只对某些条件码才有意义

**步骤**

- 自封装类型码字段
- 任选一个类型码取值，为其创建一个子类。覆写类型码类的取值函数，令其返回该类型码的字面量值
- 创建一个选择器逻辑，把类型码参数映射到新的子类
  - 如果直接继承，就用 `以工厂函数取代构造函数` 包装构造函数，选择器逻辑放在工厂函数里
  - 如果选择间接继承的方案，选择器逻辑可以保留在构造函数里
- 测试
- 针对每个类型码取指，城府上述过程。每次修改后测试
- 去除类型码字段
- 测试
- 使用 `函数下移` 和 `以多态取代条件表达式` 处理原本访问了类型码的函数
- 移除类型码的访问函数

**笔记**

使用这个重构需要考虑一个问题：是直接将「员工」类继承出「工程师」、「经理」，还是类型码下创建子类（间接）。如果是后者，则可以用 `以对象取代基本类型` 把类型码包装成「员工类别」类，然后对其使用该手法。

### 移除子类（Remove Subclass）

**简介**

子类为数据结构的多样和行为的多态提供了支持，是差异编程的好工具。但随着软件的演化，子类支持的变化可能会移除到其他地方。也有时候我们使用子类是为了应对未来的功能，结果那一天永远没有到来。

**代码展示**

```javascript
// Old
class Person {
  get genderCode() {return "X";}
}
class Male extends Person {
  get genderCode() {return "M";}
}
class Female extends Person {
  get genderCode() {return "F";}
}

// New
class Person {
  get genderCode() {return this._genderCode;}
}
```

**动机**

- 子类的存在引入了复杂性
- 子类承载的功能过少

**步骤**

- 使用 `以工厂函数取代构造函数` 把子类的构造函数包装到超类的工厂函数中
- 新建一个字段用于代表子类的类型
- 将原本针对子类的类型做判断改为用新建的类型字段
- 删除子类
- 测试

**笔记**

子类的存在是有成本的，如果它用处太少，那就不值得存在了

### 提炼超类（Extract Superclass）

**简介**

很多技术作家在谈到面向对象时，认为继承必须预先仔细规划。但很多时候，合理的继承关系是在程序演化的过程中才复现出来。这时候再把他们提炼到一起并不是一件坏事。

**代码展示**

```javascript
// Old
class Department {
  get totalAnnualCost() {...}
  get name() {...}
  get headCount() {...}
}
class Employee {
  get annualCost() {...}
  get name() {...}
  get id() {...}
}

// New
class Party {
  get name() {...}
  get annualCost() {...}
}
class Department extends Party {
  get annualCost() {...}
  get headCount() {...}
}
class Employee extends Party {
  get annualCost() {...}
  get id() {...}
}
```

**动机**

- 两个类在做相似的事

**步骤**

- 为原本的类新建一个空白的超类
- 测试
- 使用 `构造函数本体上移`，`函数上移`，`字段上移` 手法，逐一将子类的共同元素上移到超类
- 检查所有使用原本的类的客户端代码，考虑将其调整为使用超类的接口

**笔记**

通常我们可以有 `提炼类` 和这个手法使用，但一般来说这个手法会比较简单一些。


### 以委托取代子类（Replace Subclass with Delegate）

**代码展示**

```javascript
// Old
class Order {
  get daysToShip() {
    return this._warehouse.daysToShip;
  }
}
class PriorityOrder extends Order {
  get daysToShip() {
    return this._priorityPlan.daysToShip;
  }
}

// New
class Order {
  get daysToShip() {
    return (this._priorityDelegate)
      ? this._priorityDelegate.daysToShip
      : this._warehouse.daysToShip;
  }
}
class PriorityOrderDelegate {
  get daysToShip() {
    return this._priorityPlan.daysToShip
  }
}
```

**动机**

- 

**步骤**

- 如果构造函数有多个调用者，首先用 `以工厂函数取代构造函数` 包装之
- 创建一个空的类，这个类的构造函数应该接受所有子类特有的数据项，并且经常以参数的形式接受一个指回超类的引用
- 在超类中添加一个字段，用于安放委托对象
- 修改子类的创建逻辑，使其初始化上述委托字段，放入一个委托对象的实例
- 选择一个子类中的函数，将其移入委托类
- 使用 `搬移函数` 的手法搬移上述函数，不要删除类中的委托代码
- 如果被搬移的源函数还在子类之外被调用了，就把留在源类中的委托代码从子类移到超类，并在委托代码之前加上卫语句，检查委托对象的存在
- 测试
- 重复上述步骤，直到子类中的所有函数搬移到委托类
- 找到所有调用子类构造函数的地方，逐一将其改为使用超类的构造函数
- 测试
- 运用 `移除死代码` 去掉子类

**笔记**

这个手法有点奇怪，更直观的还是下面的 `以委托取代超类`。

继承给类之间引入了非常紧密的关系，使用委托可以让接口更清晰，耦合更少。

### 以委托取代超类（Replace Superclass with Delegate）

**简介**

听说 Go 语言里没有继承的概念，用委托来取代之。就是说，父类变成了一个类内普通成员，所需要的的接口我们自己实现就好。《设计模式》中有一条原则：「对象组合优于类继承」（组合即委托），我们只需要在合适的时候选择继承或者委托就好。

**代码展示**

```javascript
// Old
class List {...}
class Stack extends List {...}

// New
class Stack {
  constructor() {
    this._storage = new List();
  }
}
class List {...}
```

**动机**

- 父类的大部分对外接口我们不需要（如栈继承于列表就有很多接口是多余的）
- 不希望父类的变化影响到我们的对外接口

**步骤**

- 在子类中新建一个字段，使其引用超类的一个对象，并将这个委托引用初始化为超类的新实例
- 针对超类的每一个函数，在子类中创建一个转发函数，将调用请求转发给委托引用。每转发一块完整逻辑，都要进行测试
- 当所有超类函数都被转发函数覆写之后，就可以去掉继承关系

**笔记**

Go 语言在这方面走得很远，他们建议完全避免使用继承，但如果实际情况符合继承关系的语义条件，那么继承是一种简洁又高效的复用机制。Go 只是一个能简洁解决高并发的服务器底层开发需求的语言（甚至在敏捷开发的速度上比 Python 慢得多），没有任何语言或者解决方案是跨所有应用场景的「银弹」。建议是，优先使用继承，如果继承有问题，那么再使用 `以委托取代超类`。用趁手的语言做趁手的事，多出来的时间用来陪陪家人。

```
比如PHP专业做Web, C专门做系统, Go专门做网络服务, ASM负责调优性能, 语言学习成本又不高，一个个学就是了，未来混合语言编程是主流。。我是一个爆栈工程师, 做一个项目在N种语言中切换.  没有什么不适应的, 反而感觉很自如. 性能也能发挥最大化. 像node这一种试图通吃的语言. 只是一个失败的尝试, 时间会证明的..

作者：Zero
链接：https://www.zhihu.com/question/27172183/answer/74951148
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

<br />

---

*End.*