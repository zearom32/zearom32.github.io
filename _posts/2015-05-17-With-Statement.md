---
title: With Statement
layout: post
guid: urn:uuid:9ce9c79e-da9f-4d44-a341-7d86318239e7
tags: [Python,Pythonic]
---

# motivation
处理文件并且正确关闭：

	try:
	    f = open("example")
	    try:
	        for line in f:
	            print(line)
	    finally:
	        f.close()
	except:
	    print("error")

<!--more-->
这样做显然一点也不Pythonic，正确的做法是使用`with`

	with open("example") as f:
	    for line in f:
	        print (line)

这段代码等价于上面一段。在这里不管with之后的语句执行情况如何，f.close()一定会被调用。


# context manager
`with`语句支持runtime context defined by a context manager,一个`context manager`是指定义了`__enter__`和`__exit__`语句的对象。进入`with`时会调用对象的`__enter__`方法，并且把返回值赋给f，退出`with`语句时会调用对象的`__exit__`方法。
在上面的例子中，`open("example")`会返回一个file对象，而file对象有`__enter__`和`__exit__`方法，所以它是一个`context manager`, 然后实际上执行了`f = open("example").__enter__()`,而file的`__enter__`方法就是返回它自身。

下面就可以定义自己的`context manager`了。

	class Test():
	    def __enter__(self):
	        print("enter")
	        return "test"

	    def __exit__(self,exc_type,exc_value,traceback):
	        print("exit")
	        return


	with Test() as f:
	    print f


# @contextmanager
然而上面定义`context manager`的方法依然显得不方便。Python提供了`contextmanager`装饰器，用来方便地定义contextmanager。
在contextlib.py中可以找到如下注释

    Typical usage:

        @contextmanager
        def some_generator(<arguments>):
            <setup>
            try:
                yield <value>
            finally:
                <cleanup>

    This makes this:

        with some_generator(<arguments>) as <variable>:
            <body>

    equivalent to this:

        <setup>
        try:
            <variable> = <value>
            <body>
        finally:
            <cleanup>

被`contextmanager`装饰的generator应该只有一个yield语句，它的值会赋给`as`后的变量。我们可以重写Test

	@contextmanager
	def Test():
	    try:
	        print("enter")
	        yield "test"
	    finally:
	        print ("exit")

	with Test() as f:
	    print f


# Typical use
> Typical uses of context managers include saving and restoring various kinds of global state, locking and unlocking resources, closing opened files, etc.


# Reference
[context manager](https://docs.python.org/3/library/contextlib.html?highlight=context%20manager)  
[contextmanager type](https://docs.python.org/3/library/stdtypes.html#typecontextmanager)  
[context-managers](https://docs.python.org/3/reference/datamodel.html#context-managers)  
[with](https://docs.python.org/3/reference/compound_stmts.html#with)  
[PEP-0343](https://www.python.org/dev/peps/pep-0343/)

