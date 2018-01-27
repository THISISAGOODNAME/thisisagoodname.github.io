---
layout: post
title: "理解WebAssembly的纯文本格式"
description: "理解WebAssembly的纯文本格式"
category: WebAssembly
tags: [WebAssembly,html5]
---

&#160; &#160; &#160; &#160;在真的开始学习WebAssembly以前，我一直以为Common Lisp是个很装逼的东西。

<!-- more -->

* Table of Contents
{:toc}

&#160; &#160; &#160; &#160;为了能够直接阅读和编辑WebAssembly，wasm二进制格式提供了对应的纯文本格式(类似汇编语言提供了对应的助记符)。直接编写WebAssembly工作量其实不小，实际开发时，个人更推荐直接使用Emscripten。

# S-表达式

&#160; &#160; &#160; &#160;不论是二进制还是文本格式，WebAssembly代码中的基本单元是一个模块。在文本格式中，一个模块被表示为一个S-表达式。S-表达式是一个用来表示树的非常古老的文本格式。因此，我们可以把一个模块想象为一棵由描述了模块结构和代码的节点组成的树。不过，与一门编程语言的抽象语法树不同的是，WebAssembly包含了大部分指令列表。

&#160; &#160; &#160; &#160;首先，让我们看下S-表达式长什么样。树上的每个一个节点都有一对括号`( ... )`包围。括号内的第一个标签告诉你该节点的类型，其后跟随的是由空格分隔的属性或孩子节点列表。因此，这意味着WebAssembly的S-表达式：

```lisp
(module (memory 1) (func))
```

表示一棵根节点为“模块（module）”的树并且该树有两个孩子节点，分别是属性为1的“内存（memory）”节点和一个“函数（func）”节点。我们一会儿就会看到这些节点的含义。

## 最简单的模块

&#160; &#160; &#160; &#160;从最简单的模块就是一个空模块

```lisp
(module)
```

这个模块完全是空的，但是仍然是一个合法的模块。

&#160; &#160; &#160; &#160;如果我们现在[把该模块转换为二进制](https://developer.mozilla.org/en-US/docs/WebAssembly/Text_format_to_wasm)，我们将会看到在[二进制格式](http://webassembly.org/docs/binary-encoding/#high-level-structure)中描述的8字节的模块头：

```
0000000: 0061 736d              ; WASM_BINARY_MAGIC
0000004: 0d00 0000              ; WASM_BINARY_VERSION
```

## 向模块中添加功能

&#160; &#160; &#160; &#160;WebAssembly模块中的所有代码都是函数。函数具有下列的伪代码结构：

```lisp
( func <signature> <locals> <body> )
```

- **签名**声明函数需要的参数以及函数的返回值。
- **局部变量**类似JavaScript中的变量，但是显式的声明了类型。
- **函数体**是一个低级指令的线性列表。

# 签名和参数

&#160; &#160; &#160; &#160;签名是由一系列参数类型声明及其后面的返回值类型声明列表组成。值得注意的是：

- 没有(result)意味着函数不返回任何东西。
- 在当前版本中，最多拥有一个返回类型，但是[以后会放开这个限制到任意数量](https://webassembly.org/docs/future-features#multiple-return)。

&#160; &#160; &#160; &#160;每一个参数有一个显式声明的类型；wasm当前有四个可用类型：

- **i32**：32位整数
- **i64**：64位整数
- **f32**：32位浮点数
- **f64**：64位浮点数

&#160; &#160; &#160; &#160;一个参数写为 (param i32)，返回值类型写为(result i32)，因此，一个接受两个32位整数并返回一个64位浮点数的二进制函数会写成这样：

```lisp
(func (param i32) (param i32) (result f64) ... )
```

&#160; &#160; &#160; &#160;在签名的后面是带有类型的局部变量，例如(local i32)。总的来说，参数就是使用了调用函数传递过来的实参值进行初始化的局部变量。

# 获取和设置局部变量和参数

&#160; &#160; &#160; &#160;局部变量和参数能够被函数体使用`get_local`和`set_local`指令进行读写。

&#160; &#160; &#160; &#160;`get_local`/`set_local`指令使用数字索引来指向将被存取的条目：按照它们的声明顺序，参数在前，局部变量在后。因此，给定下面的函数：

```lisp
(func (param i32) (param f32) (local f64)
  get_local 0
  get_local 1
  get_local 2)
```

&#160; &#160; &#160; &#160;指令`get_local 0`会得到i32类型的参数，`get_local 1`会得到f32类型的参数以及`get_local 2`会得到f64类型的局部变量。

&#160; &#160; &#160; &#160;这有一个问题：使用数字索引来指向某个参数非常不直观，所以，文本格式允许你给参数、局部变量和其他大部分条目起个名字，方法就是在类型声明的前面添加一个使用美元符号（$）作为前缀的名字。

&#160; &#160; &#160; &#160;你可以像下面这样改写我们前面的签名：

```lisp
(func (param $p1 i32) (param $p2 f32) (local $loc i32) …)
```

然后，使用`get_local $p1`代替`get_local 0`，等等。（注意，当文本转换为二进制后，二进制中只包含整数。）

# 栈式机器

&#160; &#160; &#160; &#160;虽然浏览器把wasm编译为某种更高效的东西，但是，wasm的执行是以栈式机器定义的。也就是说，其基本理念是每种类型的指令都是在栈上执行特定数量的i32/i64/f32/f64类型值的入栈出栈操作。

&#160; &#160; &#160; &#160;例如，`get_local`被定义为把它读到的局部变量值压入到栈上，然后`i32.add`从栈上取出两个i32类型值（它的含义是把前面压入栈上的两个值取出来）计算它们的和（以2^32求模），最后把结果压入栈上。

&#160; &#160; &#160; &#160;当函数被调用的时候，它是从一个空栈开始的。随着函数体指令的执行，栈会逐步填满和清空。例如，在执行了下面的函数之后：

```lisp
(func (param $p i32)
  get_local $p
  get_local $p
  i32.add)
```

栈上只包含一个i32类型值——表达式 ($p + $p)的结果，该结果是由i32.add得到的。函数的返回值就是栈上留下的那个最终值。

&#160; &#160; &#160; &#160;WebAssembly验证规则确保栈准确匹配：如果你声明了(result f32)，那么，最终栈上必须包含一个f32类型值。如果没有result类型，那么栈必须是空的。

&#160; &#160; &#160; &#160;堆栈机和状态机是编程语言常见的两种实现形式，Python VM是堆栈机，lua解释器是状态机。两者在解释模式下性能差异还是蛮大的，有兴趣研究的话可以看著名论文[The Implementation of Lua 5.0](http://www.lua.org/doc/jucs05.pdf)。不过在JIT甚至AOT之后，两者性能差异并不会很大。

# 第一个函数

&#160; &#160; &#160; &#160;一个简单的执行两个整数相加的模块：

```lisp
(module
  (func (param $lhs i32) (param $rhs i32) (result i32)
    get_local $lhs
    get_local $rhs
    i32.add))
```

&#160; &#160; &#160; &#160;有很多东西都可以放在函数体里面。访问[webassembly语义手册](http://webassembly.org/docs/semantics/)获取可用操作码的完整列表。

## 调用函数

&#160; &#160; &#160; &#160;现在，我们需要调用它(一般情况是要在js中调用)。正如在一个ES2015模块里面一样，wasm函数必须通过模块里面的export语句显式地导出。

&#160; &#160; &#160; &#160;像局部变量一样，函数默认也是通过索引来区分的，但是为了方便，可以给它们起个名字。让我们由此开始——首先，在关键字func的后面增加一个美元符号开头的名字：

```lisp
(func $add … )
```

&#160; &#160; &#160; &#160;增加一个导出声明——看起来像下面这样：

```lisp
(export "add" (func $add))
```

&#160; &#160; &#160; &#160;这里的add是JavaScript中用来区别这个函数的名字，而$add则是指出模块中的哪个WebAssembly函数将会被导出。

&#160; &#160; &#160; &#160;所以，我们最终的模块（当前）看起来像下面这样：

```lisp
(module
  (func $add (param $lhs i32) (param $rhs i32) (result i32)
    get_local $lhs
    get_local $rhs
    i32.add)
  (export "add" (func $add))
)
```

&#160; &#160; &#160; &#160;把上面的模块保存到一个名叫add.wast的文件中，然后使用[wabt](https://github.com/webassembly/wabt)将其转换为名叫add.wasm的二进制文件。

&#160; &#160; &#160; &#160;接下来，我们把二进制文件加载到叫做addCode的带类型数组（获取WebAssembly字节码），编译并实例化它，然后在JavaScript中执行我们的add函数（现在，我们可以在实例的exports属性中找到add()）。

```javascript
fetchAndInstantiate('add.wasm').then(function(instance) {
   console.log(instance.exports.add(1, 2));  // "3"
});

// fetchAndInstantiate() found in wasm-utils.js
function fetchAndInstantiate(url, importObject) {
  return fetch(url).then(response =>
    response.arrayBuffer()
  ).then(bytes =>
    WebAssembly.instantiate(bytes, importObject)
  ).then(results =>
    results.instance
  );
}
```

&#160; &#160; &#160; &#160;整个实例可以在[这里](https://github.com/THISISAGOODNAME/fuckFrontEnd2017/tree/master/wasm)找到。

# 其他高级特效

## 从同一模块的其他函数调用函数

&#160; &#160; &#160; &#160;给定一个索引或名字，call指令调用相应函数。例如，下面的模块包含两个函数——一个返回值42，另一个返回调用第一个函数的结果加1之后的结果。

```lisp
(module
  (func $getAnswer (result i32)
    i32.const 42)
  (func (export "getAnswerPlus1") (result i32)
    call $getAnswer
    i32.const 1
    i32.add))
```

> i32.const只是定义一个32位整数并把它压入栈。你可以把i32替换为任何其他可用的类型并把const值修改为你想要的任何值（这里，我们把这个值设置为42）。

&#160; &#160; &#160; &#160;在这个例子中，你注意到一个`(export "getAnswerPlus1")`代码段并且它声明在第二个函数的func语句之后——这声明我们想导出这个函数以及定义导出的名字的简便方法。

&#160; &#160; &#160; &#160;从功能上来说，这与我们前面做过的那样——在函数外面即模块的其他地方包括一个独立的函数语句——是等价的。例如：

```lisp
(export "getAnswerPlus1" (func $functionName))
```

&#160; &#160; &#160; &#160;调用前面模块的JavaScript看起来像这样：

```lisp
fetchAndInstantiate('call.wasm').then(function(instance) {
  console.log(instance.exports.getAnswerPlus1());  // "43"
});
```

## 从javascript导入函数

&#160; &#160; &#160; &#160;我们已经见过JavaScript调用WebAssembly函数，但是WebAssembly如何调用JavaScript函数呢？事实上，WebAssembly对JavaScript没有任何了解，但是，它有一个可以导入JavaScript或wasm函数的通用方法。让我们看一个例子：

```lisp
(module
  (import "console" "log" (func $log (param i32)))
  (func (export "logIt")
    i32.const 13
    call $log))
```
    
&#160; &#160; &#160; &#160;WebAssembly使用了两级命名空间，所以，这里的导入语句是说我们要求从console模块导入log函数。另外，你可以看到导出的logIt函数使用我们前面介绍的call指令调用了导入的函数。

&#160; &#160; &#160; &#160;导入的函数就像普通函数一样：它们拥有一个WebAssembly验证机制会静态检查的签名并且可以被设置一个索引以及能够被命名和被调用。

&#160; &#160; &#160; &#160;JavaScript函数没有签名的概念，因此，无论导入的声明签名是什么，任何JavaScript函数都可以被传递过来。一旦一个模块声明了一个导入， [WebAssembly.instantiate()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/WebAssembly/instantiate)的调用者必须传递一个拥有相应属性的导入对象

&#160; &#160; &#160; &#160;就上面而言，我们需要一个（让我们称之为importObject的）对象并且`importObject.console.log`是一个JavaScript函数。

```javascript
var importObject = {
  console: {
    log: function(arg) {
      console.log(arg);
    }
  }
};

fetchAndInstantiate('logger.wasm', importObject).then(function(instance) {
  instance.exports.logIt();
});
```

## WebAssembly内存

&nbsp; &nbsp; &nbsp; &nbsp;上面的例子是一个相当简单的日志函数：它只是打印一个整数！要是我们想输出一个文本字符串呢？为了处理字符串及其他复杂数据类型，WebAssembly提供了内存。按照WebAssembly的定义，内存就是一个随着时间增长的字节数组。WebAssembly包含诸如i32.load和i32.store指令来实现对[线性内存](http://webassembly.org/docs/semantics/#linear-memory)的读写。

&nbsp; &nbsp; &nbsp; &nbsp;从JavaScript的角度来看，内存就是一个包括一切的大的可变大小的ArrayBuffer。从字面上来说，这也是asm.js所做的（除了它不能改变大小；参考asm.js[编程模型](http://asmjs.org/spec/latest/#programming-model)）。

&nbsp; &nbsp; &nbsp; &nbsp;因此，一个字符串就是位于这个线性内存某处的字节序列。让我们假设我们已经把一个合适的字符串字节写入到了内存中；那么，我们该如何把那个字符串传递给JavaScript呢？

&nbsp; &nbsp; &nbsp; &nbsp;关键在于JavaScript能够通过WebAssembly.Memory()接口创建WebAssembly线性内存实例并且能够通过相关的实例方法获取已经存在的内存实例（当前每一个模块实例只能有一个内存实例）。内存实例拥有一个[buffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WebAssembly/Memory/buffer)获取器，它返回一个指向整个线性内存的ArrayBuffer。

&nbsp; &nbsp; &nbsp; &nbsp;内存实例也能够增长。举例来说，在JavaScript中可以调用`Memory.grow()`方法。由于ArrayBuffer不能改变大小，所以，当增长产生的时候，当前的ArrayBuffer会被移除并且一个新的ArrayBuffer会被创建并指向新的、更大的内存。这意味着为了向JavaScript传递一个字符串，我们所需要做的就是把字符串在线性内存中的偏移量以及某种表示其长度的方法传递出去。

&nbsp; &nbsp; &nbsp; &nbsp;虽然有许多不同的方法在字符串自身当中保存字符串的长度（例如，C字符串）；但是，这里为了简单起见，我们仅仅把偏移量和长度都作为参数：

```lisp
(import "console" "log" (func $log (param i32) (param i32)))
```

&nbsp; &nbsp; &nbsp; &nbsp;在JavaScript端，我们可以使用[文本解码器API](https://developer.mozilla.org/en-US/docs/Web/API/TextDecoder)从而轻松地把我们的字节解码为一个JavaScript字符串。（这里，我们使用utf8，不过，许多其他编码也是支持的。）

```javascript
consoleLogString(offset, length) {
  var bytes = new Uint8Array(memory.buffer, offset, length);
  var string = new TextDecoder('utf8').decode(bytes);
  console.log(string);
}
```

&nbsp; &nbsp; &nbsp; &nbsp;这个谜题的最后一部分就是consoleLogString从哪里获得WebAssembly的内存（memory）实例。这里，WebAssembly给我们很大灵活性：我们既可以使用JavaScript创建一个[内存对象](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WebAssembly/Memory)并让WebAssembly模块导入这个内存或者我们让WebAssembly模块创建这个内存并把它导出给JavaScript。

&nbsp; &nbsp; &nbsp; &nbsp;为了简单起见，让我们用JavaScript创建它，然后把它导入到WebAssembly。我们的导入语句编写如下：

```lisp
(import "js" "mem" (memory 1))
```

> `1`表示导入的内存必须至少有1页内存。（WebAssembly定义一页为64KB。）

&nbsp; &nbsp; &nbsp; &nbsp;因此，让我们看一个完整的打印字符串“Hi”的模块。在一个常规的已编译的C程序，你会调用一个函数来为字符串分配一段内存，但是，因为我们正在编写自己的汇编并且我们拥有整个线性内存，所以，我们可以使用数据（data）段把字符串内容写入到一个全局内存中。数据段允许字符串字节在实例化时被写在一个指定的偏移量。而且，它与原生的可执行格式中的数据（.data）段是类似的。

我们最终的wasm模块看起来像这样：

```lisp
(module
  (import "console" "log" (func $log (param i32 i32)))
  (import "js" "mem" (memory 1))
  (data (i32.const 0) "Hi")
  (func (export "writeHi")
    i32.const 0  ;; pass offset 0 to log
    i32.const 2  ;; pass length 2 to log
    call $log))
```

> 注意上面的双分号语法，它允许在WebAssembly文件中添加注释。

&nbsp; &nbsp; &nbsp; &nbsp;现在，我们可以从JavaScript中创建一个1页的内存（Memory ）然后把它传递进去。这会在控制台输出"Hi"。

```javascript
var memory = new WebAssembly.Memory({initial:1});

var importObj = { console: { log: consoleLogString }, js: { mem: memory } };

fetchAndInstantiate('logger2.wasm', importObject).then(function(instance) {
  instance.exports.writeHi();
});
```

## WebAssembly表格

&nbsp; &nbsp; &nbsp; &nbsp;为了结束WebAssembly文本格式之旅，让我们看看最难理解的、常常令人迷惑的WebAssembly部分：表格。总的来说，表格是从WebAssembly代码中通过索引获取的可变大小的引用数组。

&nbsp; &nbsp; &nbsp; &nbsp;为了了解为什么表格是必须的，我们首先需要观察前面看到的call指令，它接受一个静态函数索引并且只调用了一个函数——但是，如果被调用者是一个运行时值呢？

- 在JavaScript中，我们总是看到：函数是一等值。
- 在C/C++中，我们看到了函数指针。
- 在C++中，我们看到了虚函数。

&nbsp; &nbsp; &nbsp; &nbsp;ebAssembly需要一种做到这一点的调用指令，因此，我们有了接受一个动态函数操作数的call_indirect指令。问题是，在WebAssembly中，当前操作数的仅有的类型是i32/i64/f32/f64。

&nbsp; &nbsp; &nbsp; &nbsp;WebAssembly可以增加一个anyfunc类型（"any"的含义是该类型能够持有任何签名的函数），但是，不幸的是，由于安全原因，这个anyfunc类型不能存储在线性内存中。线性内存会把存储的原始内容作为字节暴露出去并且这会使得wasm内容能够任意的查看和修改原始函数地址，而这在网络上是不被允许的。

&nbsp; &nbsp; &nbsp; &nbsp;解决方案是在一个表格中存储函数引用，然后作为 代替，传递表格索引——它们只是i32类型值。因此，call_indirect的操作数可以是一个i32类型索引值。

### 在wasm中定义一个表格

&nbsp; &nbsp; &nbsp; &nbsp;那么，我们该如何在表格中放置wasm函数呢？就像数据段能够用来通过字节初始化线性内存区域一样，元素（elem）段能够用来通过函数初始化表格区域：

```lisp
(module
  (table 2 anyfunc)
  (elem (i32.const 0) $f1 $f2)
  (func $f1 (result i32)
    i32.const 42)
  (func $f2 (result i32)
    i32.const 13)
  ...
)
```

- 在(table 2 anyfunc)中，数字2表示的是表格的初始大小（也就是它将存储两个引用）并且anyfunc声明的是这些引用的元素类型是“一个具有任何签名的函数”。在当前的WebAssembly版本中，这是唯一被允许的元素类型，但是在将来，更多的元素类型会加入进来。
- 函数(func)部分跟任何其他声明的wasm函数没有什么两样。她们是我们将会在表格中引用的函数（作为例子，每一个只是返回一个静态值）。值得注意的是，函数部分声明的顺序并不重要——你可以在任何地方声明你的函数然后在你的元素段（elem section）中引用它们。
- 元素段（elem section）能够将一个模块中的任意函数子集以任意顺序列入其中并允许出现重复。列入其中的函数将会被表格引用并且引用顺序是其出现的顺序。
- 元素段（elem section）中的(i32.const 0)值是一个偏移量——它需要在元素段开始的位置声明，其作用是表明函数引用是在表格中的什么索引位置开始存储的。这里我们指定的偏移量是0，表格大小是2（参考上面），因此，我们可以在索引0和1的位置填入两个引用。如果想在偏移量1的位置开始写入引用，那么，我们必须使用(i32.const 1)并且表格大小必须是3.

> 未初始化的元素会被设定一个默认的调用即抛出（throw-on-call）值。

&nbsp; &nbsp; &nbsp; &nbsp;在JavaScript中，可以创建这样一个表格实例的等价的函数调用看起来如下所示：

```javascript
function() {
  // table section
  var tbl = new WebAssembly.Table({initial:2, element:"anyfunc"});

  // function sections:
  var f1 = function() { … }
  var f2 = function() { … }

  // elem section
  tbl.set(0, f1);
  tbl.set(1, f2);
};
```

### 使用表格

&nbsp; &nbsp; &nbsp; &nbsp;现在，表格已经定义好了，我们需要用某种方法使用它。让我们使用下面的代码段来做到这一点：

```lisp
(type $return_i32 (func (result i32))) ;; if this was f32, type checking would fail
(func (export "callByIndex") (param $i i32) (result i32)
  get_local $i
  call_indirect $return_i32)
```

- `(type $return_i32 (func (param i32)))`代码块使用一个引用名字定义了一个类型。该类型被用来在后续的表格函数引用调用时进行类型检查。这里，我们声明的是该引用是一个返回值为i32类型的函数。
- 接下来，我们定义了一各导出名字为callByIndex的函数。它有一个接受i32类型的参数$i。
- 在函数里面，我们在栈顶压入一个值——该值就是传递给参数$i的值。
- 最后，我们使用call_indirect指令调用表格中的函数——这隐含的意思是$的值从栈顶出栈。最终的结果就是callByIndex函数会调用表格中的第$i个函数。
你也可以在命令调用的时候显式地声明call_indirect的参数，就像下面这样：

```lisp
(call_indirect $return_i32 (get_local $i))
```

&nbsp; &nbsp; &nbsp; &nbsp;在更高层面，像JavaScript这样更具表达力的语言，你可以设想使用一个数组（或者更有可能的是对象）来完成相同的事情。伪代码看起来像这样：`tbl[i]()`。

&nbsp; &nbsp; &nbsp; &nbsp;回到类型检查。因为WebAssembly是带有类型检查的并且anyfunc的含义是任何函数签名，所以，我们必须在调用点提供假定的被调用函数签名。这里，我们包含了一个$return_i32类型来告诉程序期望的是一个返回值为i32类型的函数。如果被调用函数没有一个匹配的签名（比如说返回值是f32类型的），那么，程序会抛出WebAssembly.RuntimeError异常。

&nbsp; &nbsp; &nbsp; &nbsp;那么，是什么把call_indirect指令和我们要是用的表格联系起来的呢？答案是，现在每一个模块实例只允许唯一一个表格存在，这也就是call_indirect指令隐式地使用的表格。在将来，当多表格被允许了，我们需要在代码行中指明一个某种形式的表格标识符：

```
call_indirect $my_spicy_table $i32_to_void
```

&nbsp; &nbsp; &nbsp; &nbsp;完整的模块看起来如下所示并且能够在我们的wasm-table.wat示例文件中找到：

```lisp
(module
  (table 2 anyfunc)
  (func $f1 (result i32)
    i32.const 42)
  (func $f2 (result i32)
    i32.const 13)
  (elem (i32.const 0) $f1 $f2)
  (type $return_i32 (func (result i32)))
  (func (export "callByIndex") (param $i i32) (result i32)
    get_local $i
    call_indirect $return_i32)
)
```

&nbsp; &nbsp; &nbsp; &nbsp;我们使用下面的JavaScript把它加载到一个网页中：

```
fetchAndInstantiate('wasm-table.wasm').then(function(instance) {
  console.log(instance.exports.callByIndex(0)); // 返回42
  console.log(instance.exports.callByIndex(1)); // 返回13
  console.log(instance.exports.callByIndex(2));
  // 返回一个错误，因为在表格中没有索引值2
});
```

> 就像内存一样，表格也能够从JavaScript中创建 (参考 [WebAssembly.Table()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WebAssembly/Table))并且能够导入和导出到其他wasm模块。
 
### 改变表格和动态链接

&nbsp; &nbsp; &nbsp; &nbsp;因为JavaScript对于函数引用有完全的存取权限，所以，从JavaScript中通过grow()、get()和set()方法能够改变表格对象。

&nbsp; &nbsp; &nbsp; &nbsp;因为表格是可变的，所以，它们能够用来实现复杂的加载时和运行时[动态链接](http://webassembly.org/docs/dynamic-linking)。当程序被动态地链接，多个实例共享相同的内存和表格。这与原生应用程序的多个.dll共享一个进程地址空间是等价的。

&nbsp; &nbsp; &nbsp; &nbsp;为了看看实际情况，我们会创建一个包含一个内存对象和一个表格对象的导入对象并且把这个导入对象传递到多个[instantiate()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WebAssembly/instantiate)调用中去。

我们的.wat看起来像这样：

shared0.wat:

```lisp
(module
  (import "js" "memory" (memory 1))
  (import "js" "table" (table 1 anyfunc))
  (elem (i32.const 0) $shared0func)
  (func $shared0func (result i32)
   i32.const 0
   i32.load)
)
```

shared1.wat:

```lisp
(module
  (import "js" "memory" (memory 1))
  (import "js" "table" (table 1 anyfunc))
  (type $void_to_i32 (func (result i32)))
  (func (export “doIt”) (result i32)
   i32.const 0
   i32.const 42
   i32.store  ;; store 42 at address 0
   i32.const 0
   call_indirect $void_to_i32)
)
```

&nbsp; &nbsp; &nbsp; &nbsp;运行逻辑如下：

1. 函数shared0func在shared0.wat中定义并存储在我们的导出表格中。
2. 该函数创建一个常量值0，然后使用i32.load指令从给定的内存索引值加载存储的值。给定的索引值为0——该指令会将之前的值出栈。所以，shared0func加载并返回存储在内存索引0处的值。
3. 在shared1.wat中，我们导出了一个名为doIt的函数——这个函数创建了两个常量值，分别为0和42，然后使用i32.store指令把给定的值存储在导入内存的给定索引处。同样的，该指令会把这些值出栈，所以，结果就是把42存储在内存索引0处。
4. 在这个函数的最后一部分，我们创建了常量值0，然后调用表格中索引0处的函数，该函数正是我们之前在shared0.wat中的使用元素代码段（elem block）存储的shared0func。
5. shared0func在被调用之后会加载我们在shared1.wat中使用i32.store指令存储在内存中的42。

&nbsp; &nbsp; &nbsp; &nbsp;上面的表达式会隐式地把这些值出栈，但是，你可以在使用指令的时候进行显式地声明。例如：

```lisp
(i32.store (i32.const 0) (i32.const 42))
(call_indirect $void_to_i32 (i32.const 0))
```

&nbsp; &nbsp; &nbsp; &nbsp;在转换为汇编之后，我们可以在JavaScript中通过下面的代码使用shared0.wasm和shared1.wasm：

```javascript
var importObj = {
  js: {
    memory : new WebAssembly.Memory({ initial: 1 }),
    table : new WebAssembly.Table({ initial: 1, element: "anyfunc" })
  }
};

Promise.all([
  fetchAndInstantiate('shared0.wasm', importObj),
  fetchAndInstantiate('shared1.wasm', importObj)
]).then(function(results) {
  console.log(results[1].exports.doIt());  // prints 42
});
```

&nbsp; &nbsp; &nbsp; &nbsp;每一个将被编译的模块都可以导入相同的内存和表格对象，这也就是共享相同的线性内存和表格的“地址空间”。

# 小结

&nbsp; &nbsp; &nbsp; &nbsp;WebAssembly的纯文本格式就是这样，不算太难，但也不太简单。想深入理解的话可以看[MDN提供的DEMO](https://github.com/mdn/webassembly-examples)。就个人观点，实际开发还是更推荐直接使用emscripten。