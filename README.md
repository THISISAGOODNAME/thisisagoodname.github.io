攻伤菊菊长的blog
==========

菊花盛开之地。


##Rake使用说明

1. Create a Post
Create posts easily via rake task:

<pre><code>
$ rake post title="Hello World"
The rake task automatically creates a file with properly formatted filename and YAML Front Matter. Make sure to specify your own title. By default, the date is the current date.

The rake task will never overwrite existing posts unless you tell it to.
</code></pre>

----------

2. Create a Page
Create pages easily via rake task:

<pre><code>
$ rake page name="about.md"
</code></pre>

Create a nested page:

<pre><code>
$ rake page name="pages/about.md"
</code></pre>

Create a page with a "pretty" path:

<pre><code>
$ rake page name="pages/about"
# this will create the file: ./pages/about/index.html
</code></pre>

The rake task automatically creates a page file with properly formatted filename and YAML Front Matter as well as includes the Jekyll Bootstrap "setup" file.

最后祝您身体健康，再见。


