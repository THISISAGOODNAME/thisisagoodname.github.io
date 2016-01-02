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

**名称**：字符串替换函数——subst。<br>
**功能**：把字串<text>中的<from>字符串替换成<to>。<br>
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

**名称**：模式字符串替换函数——patsubst。<br>
**功能**：查找<text>中的单词（单词以“空格”、“Tab”或“回车”“换行”分隔）是否符合模式<pattern>，如果匹配的话，则以<replacement>替换。这里，<pattern>可以包括通配符“%”，表示任意长度的字串。如果<replacement>中也包含“%”，那么，<replacement>中的这个“%”将是<pattern>中的那个“%”所代表的字串。（可以用“/”来转义，以“/%”来表示真实含义的“%”字符）
**返回**：函数返回被替换过后的字符串。<br>

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




