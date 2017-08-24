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

