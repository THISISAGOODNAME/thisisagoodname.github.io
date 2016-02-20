---
layout: post
title: "webidl binder"
description: "webidl binder"
category: emscripten
tags: [html5,c++,emscripten]
---

&#160; &#160; &#160; &#160;还是以一个CSS3 animation开始

<p data-height="255" data-theme-id="0" data-slug-hash="GoaboE" data-default-tab="result" data-user="THISISAGOODNAME" data-preview="true" class='codepen'>See the Pen <a href='http://codepen.io/THISISAGOODNAME/pen/GoaboE/'>Waiting</a> by 攻伤菊菊长 (<a href='http://codepen.io/THISISAGOODNAME'>@THISISAGOODNAME</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

<!-- more -->

* Table of Contents
{:toc}

# WebIDL Binder

&#160; &#160; &#160; &#160;*WebIDL Binder*是一个简单轻量的C++绑定方法，通过该方法编译过的C++代码可以在JavaScript中以和普通的JavaScript类库相同的方法调用内部的方法。

&#160; &#160; &#160; &#160;***WebIDL Binder***使用[WebIDL](https://www.w3.org/TR/WebIDL/)([中文Wiki](https://www.w3.org/html/ig/zh/wiki/WebIDL))来定义，一种专门用来粘合 C++ 和 JavaScript 的**接口定义语言**。他是 C++ 类库向 JavaScript 移植的常见选择，因为它是底层的，它也很容易进行优化。

&#160; &#160; &#160; &#160;Binder支持一个 C++ 类型的子集。虽然只是子集，但是被众多类库移植后证明是够用的 -- 甚至包括 [Box2D](https://github.com/kripken/box2d.js/#box2djs) 和 [Bullet](https://github.com/kripken/ammo.js/#ammojs) 物理引擎。

# A quick example

&#160; &#160; &#160; &#160;使用***WebIDL Binder***来粘合语言需要三步

- 创建一个WebIDL文件来描述C++接口
- 使用Binder生成C++和JavaScript的“胶水”代码
- 将“胶水”代码和工程文件一起，使用Emscripten编译

## 创建WebIDL文件

&#160; &#160; &#160; &#160;第一步是创建***WebIDL文件***来描述你将要粘合的C++类型。这个文件应该会和头文件中的信息重复，这种格式既能简化代码解析，也能简化代码的表示。

&#160; &#160; &#160; &#160;举个例子，下面的C++类

{% highlight cpp %}
class Foo {
public:
  int getVal();
  void setVal(int v);
};

class Bar {
public:
  Bar(long val);
  void doSomething();
};
{% endhighlight %}

&#160; &#160; &#160; &#160;下面的IDL文件是用来描述上面的两个类的

{% highlight cpp %}
interface Foo {
        void Foo();
        long getVal();
        void setVal(long v);
};

interface Bar {
        void Bar(long val);
        void doSomething();
};
{% endhighlight %}

&#160; &#160; &#160; &#160;IDL定义和C++源文件之间的映射关系非常明显。需要注意几件事

- IDL类定义包括了一个和接口同名返回值为`void`的函数。这个构造器可以在JavaScript中直接调用来生成实例，但是必须在IDL中定义，即使C++使用了默认构造器(比如上例中的`Foo`)
- 在WebIDL中使用的类型名不一定和他们在C++中定义时相同(举个例子，上例中`int`被映射为`long`)。更多关于映射的信息见[*WebIDL types*](#webidl-types)

> 结构体`structs`和类定义的方法相同 -- 同样都使用`interface`关键字

## 生成胶水代码

&#160; &#160; &#160; &#160;*粘合剂生成器(bindings generator)*([tools/webidl_binder.py](https://github.com/kripken/emscripten/blob/master/tools/webidl_binder.py))接收一个WebIDL文件名为输入，输出输入文件的文件名，并创建C++和JavaScript的胶水代码文件。

&#160; &#160; &#160; &#160;举个例子，为了使用IDL文件**my_classes.idl**生成胶水代码文件**glue.cpp**和**glue.js**，需要使用如下的命令

<pre><code>python tools/webidl_binder.py my_classes.idl glue</code></pre>

## 编译工程(使用胶水代码)

&#160; &#160; &#160; &#160;为了在项目中使用胶水代码文件(`glue.cpp`和`glue.js`)

* 在编译项目使用的 *emcc* 命令中添加 `--post-js glue.js`，[*post-js*](http://kripken.github.io/emscripten-site/docs/tools_reference/emcc.html#emcc-post-js)参数用来在编译时添加胶水代码
* 创建一个文件，比如**my_glue_wrapper.cpp**，用来`#include`你需要粘合的类的头文件以及 *glue.cpp* 文件，类似如下的内容

{% highlight cpp %}
#include <...> // Where "..." represents the headers for the classes we are binding.
#include <glue.cpp>
{% endhighlight %}

> 由***粘合剂生成器(bindings generator)***产生的C++胶水代码不包含他们粘合的类的头文件是因为他们无法在WebIDL文件中表示，所以需要创建一个新文件来包含头文件和胶水代码文件。如果不想创建这个新文件，可以把使用到的类的头文件引用添加到**glue.cpp**的头部，但是每次IDL文件重新编译时该过程都要重来一次

* 在最终的*emcc*命令中添加**my_glue_wrapper.cpp**文件

&#160; &#160; &#160; &#160;最终的*emcc*文件同时包括C++和JavaScript胶水文件，因为这两者其实一开始就是要一起使用的

<pre><code>./emcc my_classes.cpp my_glue_wrapper.cpp --post-js glue.js -o output.js</code></pre>

&#160; &#160; &#160; &#160;现在输出文件包含了在JavaScript中创建C++类的实例的一切

# Using C++ classes in JavaScript

&#160; &#160; &#160; &#160;当绑定完成后，C++对象可以在JavaScript直接使用，就像他们本来就是JavaScript原生对象。继续上面的例子，你可以创建`Foo`和`Bar`对象的实例并调用他们的内部方法

{% highlight javascript %}
var f = new Module.Foo();
f.setVal(200);
alert(f.getVal());

var b = new Module.Bar(123);
b.doSomething();
{% endhighlight %}

> 使用[*Module对象*](http://kripken.github.io/emscripten-site/docs/api_reference/module.html#module)来获取对象
> 虽然一般情况下对象在全局命令空间，但也存在例外(比如你使用[*closure编译器*](https://developers.google.com/closure/compiler/)来压缩和包装编译后的代码来避免污染全局命名空间)。你也可以修改Module的名字，比如使用`var MyModuleName = Module;`

&#160; &#160; &#160; &#160;JavaScript有自动垃圾回收功能，在包装的C++对象不再被使用后，会启用gc。如果C++对象不需要特别指明被清理(甚至没有析构函数)，你什么都不用做。

&#160; &#160; &#160; &#160;如果一个C++对象确实需要被清理，你需要明确调用`Module.destroy(obj)`来调用析构函数，然后删除所有引用对象，以便它可以被垃圾收集。举个例子，如果要清楚`Bar`被分配的内存：

{% highlight javascript %}
var b = new Module.Bar(123);
b.doSomething();
Module.destroy(b); // If the C++ object requires clean up
{% endhighlight %} 

> 当在JavaScript中创建C++对象时，需要显示调用C++构造器。但是没有办法得知某个JavaScript对象要被垃圾回收，所以胶水语言不能自动调用析构函数
> <br>
> 通常你需要销毁你自己创建的对象，取决于你移植的类库

# Pointers, References, Value types (Ref and Value)

&#160; &#160; &#160; &#160;C++的参数和返回类型可以是指针、引用或值类型（在堆栈上分配）。IDL文件使用不同的修饰词来代表每种情况。

&#160; &#160; &#160; &#160;在WebIDL中，没有修饰词的参数和返回值都被假定为C++中的*指针*

{% highlight cpp %}
// C++
MyClass* process(MyClass* input);
{% endhighlight %}

{% highlight cpp %}
// WebIDL
MyClass process(MyClass input);
{% endhighlight %}

&#160; &#160; &#160; &#160;引用需要修饰词`[Ref]`

{% highlight cpp %}
// C++
MyClass& process(MyClass& input);
{% endhighlight %}

{% highlight cpp %}
// WebIDL
[Ref] MyClass process([Ref] MyClass input);
{% endhighlight %}

> 如果一个引用省略了`[Ref]`修饰词，生成的胶水代码不会被编译(在尝试转换引用转换成对象时失败，因为emscripten把它当做了一个指针)

&#160; &#160; &#160; &#160;如果C++返回一个对象（而不是引用或指针）那么返回值需要修饰词`[Value]`。这将为该类分配一个静态（单例）实例，并返回它。你应该立即使用它，并在使用后不再使用。

{% highlight cpp %}
// C++
MyClass process(MyClass& input);
{% endhighlight %}

{% highlight cpp %}
// WebIDL
[Value] MyClass process([Ref] MyClass input);
{% endhighlight %}

# Const

&#160; &#160; &#160; &#160;C++中使用`const`修饰的参数和返回值类型可以在IDL中使用`[Const]`来指定。

&#160; &#160; &#160; &#160;例如，下面的代码片段显示了C++和IDL的一个函数返回一个指针常量对象。

{% highlight cpp %}
//C++
const myObject* getAsConst();
{% endhighlight %}

{% highlight cpp %}
// WebIDL
[Const] myObject getAsConst();
{% endhighlight %}

&#160; &#160; &#160; &#160;对象中使用常量指定的**属性**需要`readonly`关键字而不是`[Const]`，例如

{% highlight cpp %}
//C++
const int numericalConstant;
{% endhighlight %}

{% highlight cpp %}
// WebIDL
readonly attribute long numericalConstant;
{% endhighlight %}

&#160; &#160; &#160; &#160;这会在绑定时产生一个`get_numericalConstant()`方法，但是没有相对应的setter方法

> 多重修饰词是可以出现的。比如，一个返回常量引用的方法需要在IDL中使用`[Ref, Const]`来修饰

# Un-deletable classes (NoDelete)

&#160; &#160; &#160; &#160;如果一个类不能被删除(析构函数是私有的)，需要在IDL文件中添加`[NoDelete]`修饰

{% highlight cpp %}
[NoDelete]
interface Foo {
...
};
{% endhighlight %}

# Defining inner classes and classes inside namespaces (Prefix)

&#160; &#160; &#160; &#160;在某个命名空间(或者另一个类)中定义的类需要在IDL中使用`Prefix`来指定范围。之后当类在C++胶水语言中被音乐，都需要前缀。

&#160; &#160; &#160; &#160;举个例子，下面的IDL定义确保`Inner`类指的是`MyNameSpace::Inner`

{% highlight cpp %}
[Prefix="MyNameSpace::"]
interface Inner {
..
};
{% endhighlight %}

# Operators

&#160; &#160; &#160; &#160;可以使用`[Operator=]`来粘合C++操作符：

{% highlight cpp %}
[Operator="+="] TYPE1 add(TYPE2 x);
{% endhighlight %}

- 操作符的命名是任意的，`add`只是用来举例
- 现在仅限包含`=`的操作符，比如：`+=`、`-=`、`*=`

# enums

&#160; &#160; &#160; &#160;枚举类型在C++和IDL中定义非常类似

{% highlight cpp %}
// C++
enum AnEnum {
  enum_value1,
  enum_value2
};

// WebIDL
enum AnEnum {
  "enum_value1",
  "enum_value2"
};
{% endhighlight %}

&#160; &#160; &#160; &#160;当枚举类型在命名空间内部时，稍有复杂

{% highlight cpp %}
// C++
namespace EnumNamespace {
  enum EnumInNamespace {
        e_namespace_val = 78
  };
};

// WebIDL
enum EnumNamespace_EnumInNamespace {
  "EnumNamespace::e_namespace_val"
};
{% endhighlight %}

&#160; &#160; &#160; &#160;当枚举在一个类中定义时，枚举和类的接口定义是分离的

{% highlight cpp %}
// C++
class EnumClass {
 public:
  enum EnumWithinClass {
        e_val = 34
  };
  EnumWithinClass GetEnum() { return e_val; }

  EnumNamespace::EnumInNamespace GetEnumFromNameSpace() { return EnumNamespace::e_namespace_val; }
};



// WebIDL
enum EnumClass_EnumWithinClass {
  "EnumClass::e_val"
};

interface EnumClass {
  void EnumClass();

  EnumClass_EnumWithinClass GetEnum();

  EnumNamespace_EnumInNamespace GetEnumFromNameSpace();
};
{% endhighlight %}

# Sub-classing C++ base classes in JavaScript (JSImplementation)

&#160; &#160; &#160; &#160;*WebIDL Binder*允许C++基类在JavaScript中作为子类，在下面的IDL片段中，`[JSImplementation="Base"]`表示相关接口(`ImplJS`)是C++类`Base`的一个JavaScript实现

{% highlight cpp %}
[JSImplementation="Base"]
interface ImplJS {
        void ImplJS();
        void virtualFunc();
        void virtualFunc2();
};
{% endhighlight %}

&#160; &#160; &#160; &#160;在实行的粘合和编译过程后，你可以在JavaScript用如下方法运行接口

{% highlight cpp %}
var c = new ImplJS();
c.virtualFunc = function() { .. };
{% endhighlight %}

&#160; &#160; &#160; &#160;当C++代码中有指向`Base`实例的指针并调用`virtualFunc()`时，会到达上面定义的JavaScript代码

- 你**必须**在IDL中列举`JSImplementation`类(`ImplJS`)的全部方法，不然编译会出错
- 你也需要在IDL文件中提供`Base`类的接口定义

# Pointers and comparisons

&#160; &#160; &#160; &#160;所以绑定方法都应该接收包装好的对象(包含一个原始指针)而不是直接接收原始指针。通常你不需要处理原始指针(这些通常是简单的内存地址或者整数形式表示)。如果你需要处理原始指针，在编译好的代码中使用下面的函数会很有用

- `wrapPointer(ptr, Class)` - 接收一个原始指针(一个整数)，返回一个包装好的对象

> 如果不传递`Class`参数，会被假定为root类(很有可能不是你想要的)

- `getPointer(object)` - 返回一个原始指针
- `castObject(object, Class)` - 返回另一个类，但有相同指针的包装
- `compare(object1, object2)` - 比较两个对象的指针

> 一个特定类的指针总有一个单独的包对象。这运行你在其他地方使用普通的JavaScript代码来给对象添加数据(比如object.attribute = someData)
> <br>
> `compare()`应该代替指针的直接比较，因为有可能出现不同的包对象使用通一个个指针(一个类是另一个类的子类的时候)

# NULL 

&#160; &#160; &#160; &#160;所有返回指针、引用或者对象的绑定函数会返回一个包装好的指针。因为通过返回包，你可以获取输出并传递给另一个绑定方法，而且不需要检查参数的类型。

&#160; &#160; &#160; &#160;但是在返回`NULL`指针时你可能会疑惑。在使用粘合时，返回的指针可能是`NULL`(是一个全局实例，表示0的指针)而不是`null`(JavaScript内建的null对象)或者是数值0

# void*

&#160; &#160; &#160; &#160;`void*`类型在IDL文件中使用`VoidPtr`来表示，你也可以用`nay`类型来表示。

&#160; &#160; &#160; &#160;两者的区别是`VoidPtr`更像一个指针，可以用来获取一个包对象，而`any`更像一个32位正式(这也是Emscripten编译后的原始指针的存在形式)

# WebIDL types

&#160; &#160; &#160; &#160;WebIDL中的类型名和C++不完全相同，以下是一些常见类型的映射

|    C++    |     IDL    |
|:----------|:-----------|
|`bool`|`boolean`|
|`float`|	`float`|
|`double`	|`double`|
|`char`	|`byte`|
|`char*`|	`DOMString (represents a JavaScript string)`|
|`unsigned char`	|`octet`|
|`unsigned short int`	|`unsigned short`|
|`unsigned short`	|`unsigned short`|
|`unsigned long`|	`unsigned long`|
|`int`|	`long`|
|`void`|	`void`|
|`void*`	|`any` or `VoidPtr` (see [void*](#void))|

> WebIDL的详细文档可以参考[W3C规范](https://www.w3.org/TR/WebIDL/)
 
# Test and example code

&#160; &#160; &#160; &#160;[test_webidl](https://github.com/kripken/emscripten/tree/master/tests/webidl)，其中涵盖了绝大部分情况。

