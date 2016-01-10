---
layout: post
title: "sqlite3到sql.js编译实录"
description: "sqlite3到sql.js编译实录"
category: html5
tags: [html5,c++]
---

&#160; &#160; &#160; &#160;在正文开始之前，照例

<p data-height="500" data-theme-id="0" data-slug-hash="eJWrev" data-default-tab="result" data-user="THISISAGOODNAME" class='codepen'>See the Pen <a href='http://codepen.io/THISISAGOODNAME/pen/eJWrev/'>Times tables</a> by 攻伤菊菊长 (<a href='http://codepen.io/THISISAGOODNAME'>@THISISAGOODNAME</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

<!-- more -->

* Table of Contents
{:toc}

# SQLite简介

&#160; &#160; &#160; &#160;SQLite，是一款轻型的数据库，是遵守ACID的关系型数据库管理系统，它包含在一个相对小的C库中。它是D.RichardHipp建立的公有领域项目。它的设计目标是嵌入式的，而且目前已经在很多嵌入式产品中使用了它，它占用资源非常的低，在嵌入式设备中，可能只需要几百K的内存就够了。它能够支持Windows/Linux/Unix等等主流的操作系统，同时能够跟很多程序语言相结合，比如 Tcl、C#、PHP、Java等，还有ODBC接口，同样比起Mysql、PostgreSQL这两款开源的世界著名数据库管理系统来讲，它的处理速度比他们都快。

# 移植

&#160; &#160; &#160; &#160;得益于SQlite的精妙设计，用emscripten移植sqlite到web上非常轻松

## 下载sql.js

&#160; &#160; &#160; &#160;下载sql.js，运行

	git clone https://github.com/kripken/sql.js.git
	
这步的目的是得到[kripken](https://github.com/kripken)大神为sql.js写的Makefile，当然，这个makefile非常简单，估计大家都能写得出来

## 下载sqlite编译用源代码

&#160; &#160; &#160; &#160;到sqlite的[下载页面](http://sqlite.org/download.html)，下载sqlite amalgamation，我下载的是[sqlite-amalgamation-3100000.zip](http://sqlite.org/2016/sqlite-amalgamation-3100000.zip),sqlite合并版c源码只有一个.c文件和一个.h文件，非常便于作为嵌入式子系统嵌入到自己的程序中。

## 放置源代码到正确位置

&#160; &#160; &#160; &#160;把下载到的sqlite-amalgamation压缩包解压，复制`sqlite3.c`和`sqlite3.h`两个文件到sql.js的c文件夹下

## 编译

&#160; &#160; &#160; &#160;执行如下命令

{% highlight bash %}
# 清理
make clean

# 生成sql.js
make all

# 生成sql-debug.js
make debug

# 生成worker.sql.js(需要coffee)
make worker
{% endhighlight %}

&#160; &#160; &#160; &#160;生成的sql.js文件在js文件夹下

# sql.js的使用

## node.js中使用

{% highlight javascript %}
var sql = require('sql.js');
// or sql = window.SQL if you are in a browser

// Create a database
var db = new sql.Database();
// NOTE: You can also use new sql.Database(data) where
// data is an Uint8Array representing an SQLite database file

// Execute some sql
sqlstr = "CREATE TABLE hello (a int, b char);";
sqlstr += "INSERT INTO hello VALUES (0, 'hello');"
sqlstr += "INSERT INTO hello VALUES (1, 'world');"
db.run(sqlstr); // Run the query without returning anything

var res = db.exec("SELECT * FROM hello");
/*
[
    {columns:['a','b'], values:[[0,'hello'],[1,'world']]}
]
*/

// Prepare an sql statement
var stmt = db.prepare("SELECT * FROM hello WHERE a=:aval AND b=:bval");

// Bind values to the parameters and fetch the results of the query
var result = stmt.getAsObject({':aval' : 1, ':bval' : 'world'});
console.log(result); // Will print {a:1, b:'world'}

// Bind other values
stmt.bind([0, 'hello']);
while (stmt.step()) console.log(stmt.get()); // Will print [0, 'hello']

// free the memory used by the statement
stmt.free();
// You can not use your statement anymore once it has been freed.
// But not freeing your statements causes memory leaks. You don't want that.

// Export the database to an Uint8Array containing the SQLite database file
var binaryArray = db.export();
{% endhighlight %}

## 网页中使用

{% highlight html %}
<script src='js/sql.js'></script>
<script>
    //Create the database
    var db = new SQL.Database();
    // Run a query without reading the results
    db.run("CREATE TABLE test (col1, col2);");
    // Insert two rows: (1,111) and (2,222)
    db.run("INSERT INTO test VALUES (?,?), (?,?)", [1,111,2,222]);

    // Prepare a statement
    var stmt = db.prepare("SELECT * FROM test WHERE col1 BETWEEN $start AND $end");
    stmt.getAsObject({$start:1, $end:1}); // {col1:1, col2:111}

    // Bind new values
    stmt.bind({$start:1, $end:2});
    while(stmt.step()) { //
        var row = stmt.getAsObject();
        // [...] do something with the row of result
    }
</script>
{% endhighlight %}

也可以参照[这个](http://kripken.github.io/sql.js/GUI)样例

## 从server上读取数据库

{% highlight javascript %}
var xhr = new XMLHttpRequest();
xhr.open('GET', '/path/to/database.sqlite', true);
xhr.responseType = 'arraybuffer';

xhr.onload = function(e) {
  var uInt8Array = new Uint8Array(this.response);
  var db = new SQL.Database(uInt8Array);
  var contents = db.exec("SELECT * FROM my_table");
  // contents is now [{columns:['col1','col2',...], values:[[first row], [second row], ...]}]
};
xhr.send();
{% endhighlight %}

具体使用方法见[这里](https://github.com/kripken/sql.js/wiki/Load-a-database-from-the-server)

## 用协程的方式运行sql.js

{% highlight html %}
<script>
var worker = new Worker("js/worker.sql.js"); // You can find worker.sql.js in this repo
worker.onmessage = function() {
    console.log("Database opened");
    worker.onmessage = function(event){
        console.log(event.data); // The result of the query
    };
    worker.postMessage({
        id: 2,
        action: 'exec',
        sql: 'SELECT * FROM test'
    });
};

worker.onerror = function(e) {console.log("Worker error: ", e)};
worker.postMessage({
    id:1,
    action:'open',
    buffer:buf, /*Optional. An ArrayBuffer representing an SQLite Database file*/
});
</script>
{% endhighlight %}