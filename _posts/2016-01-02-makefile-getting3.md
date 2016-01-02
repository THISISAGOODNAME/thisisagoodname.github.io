---
layout: post
title: "makefile简易教程(3)"
description: "makefile简易教程(2)"
category: c++
tags: [c++,自动化工具]
---
* Table of Contents
{:toc}



<!-- more -->

# 7 使用函数

&#160; &#160; &#160; &#160;在Makefile中可以使用函数来处理变量，从而让我们的命令或是规则更为的灵活和具有智能。make所支持的函数也不算很多，不过已经足够我们的操作了。函数调用后，函数的返回值可以当做变量来使用。

## 7.1 函数的调用语法

&#160; &#160; &#160; &#160;函数调用，很像变量的使用，也是以“$”来标识的，其语法如下：

{% highlight makefile %}
$(<function> <arguments>)
{% endhighlight %}

或是

{% highlight makefile %}
${<function> <arguments>}
{% endhighlight %}
    
这里，&lt;function>就是函数名，make支持的函数不多。&lt;arguments>是函数的参数，参数间以逗号“,”分隔，而函数名和参数之间以“空格”分隔。函数调用以“$”开头，以圆括号或花括号把函数名和参数括起。感觉很像一个变量，是不是？函数中的参数可以使用变量，为了风格的统一，函数和变量的括号最好一样，如使用“$(subst a,b,$(x))”这样的形式，而不是“$(subst a,b,${x})”的形式。因为统一会更清楚，也会减少一些不必要的麻烦。

还是来看一个示例：

{% highlight makefile %}
comma:= ,
empty:=
space:= $(empty) $(empty)
foo:= a b c
bar:= $(subst $(space),$(comma),$(foo))
{% endhighlight %}

在这个示例中，$(comma)的值是一个逗号。$(space)使用了$(empty)定义了一个空格，$(foo)的值是“a b c”，$(bar)的定义用，调用了函数“subst”，这是一个替换函数，这个函数有三个参数，第一个参数是被替换字串，第二个参数是替换字串，第三个参数是替换操作作用的字串。这个函数也就是把$(foo)中的空格替换成逗号，所以$(bar)的值是“a,b,c”。

## 7.2 字符串处理函数

{% highlight makefile %}
$(subst <from>,<to>,<text>)
{% endhighlight %}

**名称**：字符串替换函数——subst。

**功能**：把字串&lt;text>中的&lt;from>字符串替换成&lt;to>。

**返回**：函数返回被替换过后的字符串。

**示例**：

{% highlight makefile %}
$(subst ee,EE,feet on the street)
{% endhighlight %}        
        
        
&#160; &#160; &#160; &#160;把“feet on the street”中的“ee”替换成“EE”，返回结果是“fEEt on the strEEt”。

---

{% highlight makefile %}
$(patsubst <pattern>,<replacement>,<text>)
{% endhighlight %}

**名称**：模式字符串替换函数——patsubst。

**功能**：查找&lt;text>中的单词（单词以“空格”、“Tab”或“回车”“换行”分隔）是否符合模式&lt;pattern>，如果匹配的话，则以&lt;replacement>替换。这里，&lt;pattern>可以包括通配符“%”，表示任意长度的字串。如果&lt;replacement>中也包含“%”，那么，&lt;replacement>中的这个“%”将是&lt;pattern>中的那个“%”所代表的字串。（可以用“/”来转义，以“/%”来表示真实含义的“%”字符）

**返回**：函数返回被替换过后的字符串。

**示例**：

{% highlight makefile %}
$(patsubst %.c,%.o,x.c.c bar.c)
{% endhighlight %}
        

&#160; &#160; &#160; &#160;把字串“x.c.c bar.c”符合模式[%.c]的单词替换成[%.o]，返回结果是“x.c.o bar.o”

**备注**：

&#160; &#160; &#160; &#160;这和我们前面“变量章节”说过的相关知识有点相似。如：

&#160; &#160; &#160; &#160;`(var:<pattern>=<replacement>)`
&#160; &#160; &#160; &#160;相当于`“$(patsubst <pattern>,<replacement>,$(var))`，

&#160; &#160; &#160; &#160;而`$(var: <suffix>=<replacement>)`则相当于`$(patsubst %<suffix>,%<replacement>,$(var))`。

&#160; &#160; &#160; &#160;例如有：objects = foo.o bar.o baz.o，
&#160; &#160; &#160; &#160;那么，`$(objects:.o=.c)`和`$(patsubst %.o,%.c,$(objects))`是一样的。

---

{% highlight makefile %}
$(strip <string>)
{% endhighlight %}

**名称**：去空格函数——strip。

**功能**：去掉&lt;string>字串中开头和结尾的空字符。

**返回**：返回被去掉空格的字符串值。

**示例**：
        
{% highlight makefile %}
$(strip a b c )
{% endhighlight %}	

&#160; &#160; &#160; &#160;把字串“a b c ”去到开头和结尾的空格，结果是“a b c”。

---

{% highlight makefile %}
$(findstring <find>,<in>)
{% endhighlight %}	


**名称**：查找字符串函数——findstring。

**功能**：在字串&lt;in>中查找&lt;find>字串。

**返回**：如果找到，那么返回&lt;find>，否则返回空字符串。

**示例**：

{% highlight makefile %}
$(findstring a,a b c)
$(findstring a,b c)
{% endhighlight %}

&#160; &#160; &#160; &#160;第一个函数返回“a”字符串，第二个返回“”字符串（空字符串）

---

{% highlight makefile %}
$(filter <pattern...>,<text>)
{% endhighlight %}

**名称**：过滤函数——filter。

**功能**：以&lt;pattern>模式过滤&lt;text>字符串中的单词，保留符合模式&lt;pattern>的单词。可以有多个模式。

**返回**：返回符合模式&lt;pattern>的字串。

**示例**：

{% highlight makefile %}
sources := foo.c bar.c baz.s ugh.h
foo: $(sources)
        cc $(filter %.c %.s,$(sources)) -o foo
{% endhighlight %}
        

&#160; &#160; &#160; &#160;$(filter %.c %.s,$(sources))返回的值是“foo.c bar.c baz.s”。

---

{% highlight makefile %}
$(filter-out <pattern...>,<text>)
{% endhighlight %}

**名称**：反过滤函数——filter-out。

**功能**：以&lt;pattern>模式过滤&lt;text>字符串中的单词，去除符合模式&lt;pattern>的单词。可以有多个模式。

**返回**：返回不符合模式&lt;pattern>的字串。

**示例**：

{% highlight makefile %}
objects=main1.o foo.o main2.o bar.o
mains=main1.o main2.o
{% endhighlight %}
        
        
&#160; &#160; &#160; &#160;$(filter-out $(mains),$(objects)) 返回值是“foo.o bar.o”。
        
---        

{% highlight makefile %}
$(sort <list>)
{% endhighlight %}        

**名称**：排序函数——sort。

**功能**：给字符串&lt;list>中的单词排序（升序）。

**返回**：返回排序后的字符串。

**示例**：$(sort foo bar lose)返回“bar foo lose” 。

**备注**：sort函数会去掉&lt;list>中相同的单词。

---

{% highlight makefile %}
$(word <n>,<text>)
{% endhighlight %}  

**名称**：取单词函数——word。

**功能**：取字符串&lt;text>中第&lt;n>个单词。（从一开始）

**返回**：返回字符串&lt;text>中第&lt;n>个单词。如果&lt;n>比&lt;text>中的单词数要大，那么返回空字符串。

**示例**：$(word 2, foo bar baz)返回值是“bar”。

---

{% highlight makefile %}
$(wordlist <s>,<e>,<text>) 
{% endhighlight %}  

**名称**：取单词串函数——wordlist。

**功能**：从字符串&lt;text>中取从&lt;s>开始到&lt;e>的单词串。&lt;s>和&lt;e>是一个数字。

**返回**：返回字符串&lt;text>中从&lt;s>到&lt;e>的单词字串。如果&lt;s>比&lt;text>中的单词数要大，那么返回空字符串。如果&lt;e>大于&lt;text>的单词数，那么返回从&lt;s>开始，到&lt;text>结束的单词串。

**示例**： $(wordlist 2, 3, foo bar baz)返回值是“bar baz”。
    
----    
  
{% highlight makefile %}
$(words <text>)
{% endhighlight %}     
    
**名称**：单词个数统计函数——words。

**功能**：统计&lt;text>中字符串中的单词个数。

**返回**：返回&lt;text>中的单词数。

**示例**：$(words, foo bar baz)返回值是“3”。

**备注**：如果我们要取&lt;text>中最后的一个单词，我们可以这样：$(word $(words &lt;text>),&lt;text>)。

---

{% highlight makefile %}
$(firstword <text>)
{% endhighlight %}

**名称**：首单词函数——firstword。

**功能**：取字符串&lt;text>中的第一个单词。

**返回**：返回字符串&lt;text>的第一个单词。

**示例**：$(firstword foo bar)返回值是“foo”。

**备注**：这个函数可以用word函数来实现：$(word 1,&lt;text>)。

---

&#160; &#160; &#160; &#160;以上，是所有的字符串操作函数，如果搭配混合使用，可以完成比较复杂的功能。这里，举一个现实中应用的例子。我们知道，make使用“VPATH”变量来指定“依赖文件”的搜索路径。于是，我们可以利用这个搜索路径来指定编译器对头文件的搜索路径参数CFLAGS，如：

{% highlight makefile %}
override CFLAGS += $(patsubst %,-I%,$(subst :, ,$(VPATH)))
{% endhighlight %}
    

&#160; &#160; &#160; &#160;如果我们的“$(VPATH)”值是“src:../headers”，那么“$(patsubst %,-I%,$(subst :, ,$(VPATH)))”将返回“-Isrc -I../headers”，这正是cc或gcc搜索头文件路径的参数。