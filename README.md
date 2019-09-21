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




---

*To be continue*
