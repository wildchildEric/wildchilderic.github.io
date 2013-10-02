---
layout: post
title: "Ruby中Block,Proc,Lambda区别(译)"
date: 2013-10-01 22:52
comments: true
categories: [Ruby]
---

###block,proc,lambda是什么？

行话：闭包在Ruby中的体现。
</br>
白话：我们想复用的代码的组织方式。

``` ruby Block Examples
[1,2,3].each { |x| puts x*2 }   # block is in between the curly braces

[1,2,3].each do |x|
  puts x*2                    # block is everything between the do and end
end

# Proc Examples             
p = Proc.new { |x| puts x*2 } 
[1,2,3].each(&p)              # The '&' tells ruby to turn the proc into a block 

proc = Proc.new { puts "Hello World" }
proc.call                     # The body of the Proc object gets executed when called

# Lambda Examples            
lam = lambda { |x| puts x*2 } 
[1,2,3].each(&lam)

lam = lambda { puts "Hello World" }
lam.call

# proc 和 lambda 的简写形式
p = proc{ |x| puts x*2 }
lam = ->(x) {puts x*2}
```

{% blockquote rubymonk.com http://rubymonk.com/learning/books/4-ruby-primer-ascent/chapters/18-blocks/lessons/64-blocks-procs-lambdas Blocks, Procs, and Lambdas %}
译注：
-> 写法是 Kernel#lambda 的简短形式。
Kernel#proc工厂方法和Proc.new是一样的。注意proc与 ->,yield不同,它是个方法。
{% endblockquote %}

这些代码看起来非常相似，但其中有些细微的区别，我们接下来会讲述。

###Block和Proc区别

####1.proc是对象，而block不是

一个proc是Proc类的实例。
``` ruby
p = Proc.new { puts "Hello World" }
```

这使得我们可以在proc上调用它的方法，赋值给变量。proc也可以返回他们自己。
``` ruby
p.call  # prints 'Hello World'
p.class # returns 'Proc'
a = p   # a now equals p, a Proc instance
p       # returns a proc object '#<Proc:0x007f96b1a60eb0@(irb):46>'
```

相反，block只是方法调用语法的一部分。单独存在的block没有意义且只能存在于方法调用的参数列表里。
``` ruby
{ puts "Hello World"}       # syntax error  
a = { puts "Hello World"}   # syntax error
[1,2,3].each {|x| puts x*2} # only works as part of the syntax of a method call
```

####2.在参数列表中，block只能最多出现一次
相反，你可以向方法传递多个proc
``` ruby
def multiple_procs(proc1, proc2)
  proc1.call
  proc2.call
end

a = Proc.new { puts "First proc" }
b = Proc.new { puts "Second proc" }

multiple_procs(a,b)
```

###Proc和Lambda的区别
在我深入他们区别之前，需要强调的是他们都是Proc对象。
``` ruby
proc = Proc.new { puts "Hello world" }
lam = lambda { puts "Hello World" }

proc.class # returns 'Proc'
lam.class  # returns 'Proc'
```
然而lambda是一种不同的proc，对象返回的时候我们可以觉察到这细微的差别：
``` ruby
proc   # returns '#<Proc:0x007f96b1032d30@(irb):75>'
lam    # returns '<Proc:0x007f96b1b41938@(irb):76 (lambda)>'
```
'(lambda)'记号提醒我们他们很相似，但也有不同，下面就是一些关键的区别

####1.Lambda检查参数的个数，而proc则不
``` ruby
lam = lambda { |x| puts x }    # creates a lambda that takes 1 argument
lam.call(2)                    # prints out 2
lam.call                       # ArgumentError: wrong number of arguments (0 for 1)
lam.call(1,2,3)                # ArgumentError: wrong number of arguments (3 for 1)
```
相反,proc并不关心参数个数是否正确
``` ruby
proc = Proc.new { |x| puts x } # creates a proc that takes 1 argument
proc.call(2)                   # prints out 2
proc.call                      # returns nil
proc.call(1,2,3)               # prints out 1 and forgets about the extra arguments
```
如上所示，在参数个数错误的情况下调用proc，并没有发生错误。如果proc需要一个参数，但是没有参数传入，这样的情况下proc会返回nil值；如果参数个数多余proc所需参数个数，则proc会忽略多余的参数。

####2.Lambda和proc以不同方式处理'return'关键字
在lambda中的'return'返回到调用lambda的地方
``` ruby
def lambda_test
  lam = lambda { return }
  lam.call
  puts "Hello world"
end

lambda_test                 # calling lambda_test prints 'Hello World'
```
在proc中的'return'返回到调用包含proc调用的方法的地方
``` ruby
def proc_test
  proc = Proc.new { return }
  proc.call
  puts "Hello world"
end

proc_test                 # calling proc_test prints nothing
```

###那么什么是闭包（closure）？
行话：带有引用环境的函数或者函数引用。不同于单纯的函数，闭包允许访问非本地变量，甚至在其作用域外。-维基百科
</br>
白话：类似一个手提箱，是一组这样的代码： 在其被打开（被调用）时，包含了所有当时打包的（创建的）。
``` ruby Example of Proc objects preserving local context
def counter
  n = 0
  return Proc.new { n+= 1 }
end

a = counter
a.call            # returns 1
a.call            # returns 2

b = counter
b.call            # returns 1

a.call            # returns 3
```

###背景部分1：Lambda微积分和匿名函数
Lambda得名于一种1930年代用于研究基础数学的微积分，Lambda微积分帮助简化可计算方程式的语意从而易于被学习。相关度最高的简化是使方程式匿名，也就是没有显式的基于方程式名字。
```ruby
sqsum(x,y) = x*x + y*y  #<-- normal function
(x,y) -> x*x + y*y #<-- anonymous function
```
一般而言，在编程中，术语lambda指匿名函数，这些匿名函数在一些编程语言中很常见且是显式的（如javascript），而在另外一些语言中则比较隐秘（如Ruby）。

###背景部分2：Proc名字的由来
Proc是procedure（程序,过程）的缩写，既是用来执行特定任务而被打包成单元的一组指令。在不同的语言中，可能被称作function，routine，method或者unit。他们在一个程序中通常会被调用多次。

###区别总结

1.proc是对象，而block不是</br>
2.在参数列表中，最多只能有一个block</br>
3.lambda检查参数个数，proc不检查</br>
4.lambda和proc处理‘return’的方式不同</br>

原文:[What Is the Difference Between a Block, a Proc, and a Lambda in Ruby?](http://awaxman11.github.io/blog/2013/08/05/what-is-the-difference-between-a-block/)





