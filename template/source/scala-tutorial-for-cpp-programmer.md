title: 面向c++程序员的scala教程
date: 2016-03-20 22:49:00 +0800
update: 2016-03-20 22:49:00  +0800
author: me
#cover: -/images/default.png
tags:
    - C++
    - Scala 
    
---

最近在研究finagle，打算先从scala语言开始学习。本文不能作为scala的入门教程，内容也不尽完善和准确，只是作为个人学习笔记之用。

<!--more-->

## scala观点
- [Scala语言设计有哪些缺陷?](https://www.zhihu.com/question/28573046)
- [为什么说 Scala 是 JVM 上的 C++？](https://www.zhihu.com/question/27332932)
- [为啥 Erlang 没有像 Go、Scala 语言那样崛起？](https://www.zhihu.com/question/38032439)
- [Scala 是一门怎样的语言，具有哪些优缺点？](https://www.zhihu.com/question/19748408)
- [有趣的 Scala 语言: 函数成了一等公民](http://www.ibm.com/developerworks/cn/java/j-lo-funinscala3/index.html)

一个有趣的观点
 ![图片来自zhihu.com，侵删](-/images/scala-view-point.png)

## Scala学习资源
- [Scala与Clojure函数式编程模式：Java虚拟机高效编程](http://product.dangdang.com/1349099431.html)
- [深入理解Scala](http://product.dangdang.com/23630366.html)
- [Scala课堂](http://twitter.github.io/scala_school/zh_cn/index.html)
- [Scala语言官方站点](http://docs.scala-lang.org/)
- [Effective Scala](http://twitter.github.io/effectivescala/index-cn.html)
- [官方编码规范](http://docs.scala-lang.org/style/)
- [Scala api reference](http://www.scala-lang.org/api/current/#package)

## 构建基本的scala环境

### windows

由于的代理服务器不是很稳定，所以使用typesafe的[activator](http://www.lightbend.com/activator/download)作为开发构建工具，兼容sbt的所有命令

### Mac

typesafe会搜寻如下文件夹进行执行

- Sources in the base directory
- Sources in src/main/scala or src/main/java
- Tests in src/test/scala or src/test/java
- Data files in src/main/resources or src/test/resources
- jars in lib

### IDE
推荐IntelliJ Idea，支持sbt构建工具
[下载地址](https://www.jetbrains.com/idea/download/)

### activator环境测试
输入`activator shell`进入shell模式，`console`进入scala的repl。

建立src\main\scala\helloworld.scala文件，编写测试代码

``` scala
object Hi {
  def main(args: Array[String]) = println("Hello world!")
}
```

在shell模式下执行`run`，输出"Hello world!"，基本scala编译环境准备完成，这里不对sbt作过多的学习。

## 基础部分

### 内置类型
![scala type hierarchy](-/images/scala-type-hierarchy.jpg)

### 定义可变变量
``` scala
var variableType : String = "123"
```

### 定义不可变变量
``` scala
val constVariableType : Int = 123
```

### 函数
不带参数时括号可以省略，返回类型可以被推导时可以省略返回值类型。
``` scala
def noneParamFunction = "123"
def noneParamWithReturnType : String= "123"
def onePramWithReturnType(i : Int) = i + 1
def functionDefinationSpanTwoLine(i : Int) = {
   val a = i + 1
   a + 1
}
```

返回值为空的函数
``` scala
def nullReturnValueFun={} //f:Uint`
```

使用`=>`创建匿名函数，匿名不需要制定返回值。
``` scala
(i : Int) => i + 1
```

### 注释
和C++一样，支持//和/*...*/

### 流程控制
直接把语法定义搬过来

    `if' `(' Expr `)' {nl} Expr [[semi] `else' Expr]
    `while' `(' Expr `)' {nl} Expr
    `try' (`{' Block `}' | Expr) [`catch' `{' CaseClauses `}'] [`finally' Expr]
    `do' Expr [semi] `while' `(' Expr ')'
    `for' (`(' Enumerators `)' | `{' Enumerators `}') {nl} [`yield'] Expr
    `throw' Expr
    `return' [Expr]

### 关键字
> abstract case catch class def  
do else extends false final  
finally for forSome if implicit  
import lazy macro match new   
null object override package private   
protected return sealed super this   
throw trait try true type   
val var while with yield   
_ : = => <- <: <% >: # @  

***注意：***在scala中访问java标识符含有scala关键字式用``进行转义。

### 表达式
>Scala中（几乎）一切都是表达式

### 包Package

#### 声明包
``` scala
package myFirstPackage
```

#### 支持嵌套
``` scala
package com {
    package tizzybec {
        package toy {
            ...
        }    
    }
}
```

#### 引用包

引用制定成员
``` scala
import java.awt.Color
```

引用所有成员
``` scala
import java.awt._
```

引用部分成员
``` scala
import java.awt.{Color, Font}
```

给成员别名
``` scala
import java.util.{HashMap => JavaHashMap}
```

使用别名和部分成员可以达到吟唱部分包的目的
``` scala
import java.util.{HashMap => _, _} 
```

对于当前包宇引入包内成员名字重名的情况使用`_root_`指向包的根部来引用当前包内的制定成员

### 对象Class

class定义与java相似
``` scala
class myFirstScalaClass {}
```

单例模式支持
``` scala
object SingletonClass
```

私有变量，同名的class和object互为友元
``` scala
class ObjectHasPrivateMember {
    import ObjectHasPrivateMember ._

    def privateMember = realPrivateMember
}

object ObjectHasPrivateMember {
    private def readPrivateMember = 12
}
```

### 异常支持
异常支持应该是为了兼容java库的使用，除非使用java带异常函数，个人很少使用异常

``` scala
val result: Int = try {
        remoteCalculatorService.add(1, 2)
    } catch { 
         case e: ServerIsDownException => { log.error(e, "the remote calculator service is unavailable. should have kept your trusty HP.") 0 }
    } finally { 
        remoteCalculatorService.close()
    }
```
scala的异常是表达式级别的，支持finally，支持模式匹配

### 与java的互操作性
如果对scala生成的代码有疑问，使用JDK自带的javap反编译工具进行查看，由于我对java字节码不是很熟悉，这里就不深入了

具体操作规则略。

## 特性trait

特性和java/C++中的虚函数属于同等概念，trait定义的接口可实现也可以不实现

trait可以从class继承，通过with进行mixin（混入），从scala-lang引入一个例子

``` scala
abstract class AbsIterator{
    type T
    def hasNext:Boolean
    defnext: T
 }

trait RichIterator extends AbsIterator {
    defforeach(f: T => Unit) { while(hasNext) f(next) }
}

class StringIterator(s: String) extends AbsIterator {
    type T = Char
    private var i =0
    def hasNext = i < s.length()
    def next = { val ch = s charAt i; i +=1; ch }
}

object StringIteratorTest {
    def main(args:Array[String]){
        class Iter extendsStringIterator(args(0)) with RichIterator
        val iter = newIter
        iter foreach println
    }
}
```

通过使用`with RichIterator`，Iter 混入了RichIterator的行为，也就是mixin-class composition，对于不支持多重继承的语言，mixin能够很好突破这个限制，对mixin理解为内联还是ducktype的语法糖呢？

延伸阅读：[Mixin是什么概念](https://www.zhihu.com/question/20778853)

## 集合collection

scala对集合的支持非常完善，很适合进行大数据的处理。集合主要分为四类

- 链表List
- 集Set
- 序列Seq
- 映射map

这几类是基类，在scala中有更多的用途的集合。

所有集合都拥有以下特质

- 可遍历性，Traversable特质，定义了map、foreach、find、filter、partition和groupBy等函数
- 可迭代性，Iterable特质，定义了hasNext和next等函数

集合严格区分mutable和immutable，不可变特性是函数式编程中经常使用，可变状态会带来线程安全问题

## 注解

scala在注解的使用上基本和java保持一致，这个特性是在JDK1.5以后引入的，注解相当于代码的元数据，scala常用的注解包括：

- cloneable
- inline
- native
- remote
- serializable
- throws
- transient
- uncheked
- volatile

这篇文章[Scala基础之注解(annotation)](http://roadtopro.me/scala/annotation/)讲了常用注解的使用

用上注解就有点c++的味道了，比起c++，在写法上更具一致性，像Deprecated，c++里只能只能以来各个编译器的隐含特性，tailrec支持尾递归优化就实在是太赞了，还有像transient对属性进行注解能更加精准地控制序列化的过程，c++里面撸reflection和serialization可就全靠hack手段了（全是宏和模板）

## 函数式编程
### 柯里化
scala对currying的支持非常好，直接定义参数链就可以：

``` scala
def curryingFunc(i: Int)(j :  String)
```

### 函数组合

组合函数直接内置compose关键字进行支持

``` scala
def f(s String): String = "f(" + s + ")"
def g(s: String): String= "g(" + s + ")"
def fComposeG = f _ compose g _
fComposeG ("x")  //return f(g(x))
```

scala提供一个和compose相反调用顺序的函数andThen ，改写上面的例子

``` scala
def f(s String): String = "f(" + s + ")"
def g(s: String): String= "g(" + s + ")"
def fAndThenG = f _ andThen g _
fAndThenG ("x")  //return g(f(x))
```

### 偏函数（partial function）

先看看函数和偏函数的差异：
 
> 对给定的输入参数类型，函数可接受该类型的任何值.
> 对给定的输入参数类型，偏函数只能接受该类型的某些特定的值

偏函数支持调用isDefinedAt方法查询是够支持给定参数，偏函数的一个特例就是case语句，case本身就是偏函数的一个子类

使用orElse可以对偏函数进行组合，语句从第一个偏函数开始匹配直到匹配输入，或者到达最后一个orElse

``` scala
val partial = one orElse two orElse three orElse wildchar
```

上述语句先检查one是够接受参数，然后是two，然后是three，如果均没有匹配则匹配最后的通配符wildchar

偏函数是函数的子类型，任何接受函数参数的地方均接受偏函数

## 泛型

### 类型推导
Scala有秩1多态性(未查到)，下面的代码无法编译通过

``` scala
def foo[A](f: A => List[A], i: Int) = f(i)
```

如果拿C++改写，可能是这样的（徒手码的）

``` scala
template <typename A>
std::list<A> foo(std::list<A> (const *f)(A), const A &i) {
    return f(i);
}
```

C++编译器能够正常推导所有类型

在scala school中有这么一句话：

> 在Scala中所有类型推断是局部的。Scala一次分析一个表达式。

我们可以理解为scala编译器分别对`f: A => List[A]`和`i: Int`进行了推导，但是没有对f(i)进行联合推导，这点从C++过来的多多少少有些不适应。

## 模式匹配
使用函数式语言，模式匹配的支持确实给代码的编写带来很多便利，scala通过match关键字进行模式匹配，看一个例子
``` scala
val times = 1
times match {
  case 1 => "one"
  case 2 => "two"
  case _ => "some other number"
}
```
### 类型匹配支持
``` scala
def bigger(o: Any): Any = {
  o match {
    case i: Int if i < 0 => i - 1
    case i: Int => i + 1
    case d: Double if d < 0.0 => d - 0.1
    case d: Double => d + 0.1
    case text: String => text + "s"
  }
}
类成员匹配，和if语句差不过
``` scala
def calcType(calc: Calculator) = calc match {
  case _ if calc.brand == "hp" && calc.model == "20B" => "financial"
  case _ if calc.brand == "hp" && calc.model == "48G" => "scientific"
  case _ if calc.brand == "hp" && calc.model == "30B" => "business"
  case _ => "unknown"
}
```
### 样本类（Case classes）匹配
``` scala
case class Calculator(brand: String, model: String)
val hp20b = Calculator("hp", "20B")
val hp30b = Calculator("hp", "30B")
def calcType(calc: Calculator) = calc match {
    case Calculator("hp", "20B") => "financial"
    case Calculator("hp", "48G") => "scientific"
    case Calculator("hp", "30B") => "business"
    case Calculator(ourBrand, ourModel) => "Calculator: %s %s is of unknown type".format(ourBrand, ourModel)
}
```
## 协变，逆变
关于逆变和协变，可以参考以下博客进行学习
[Java中的逆变与协变](http://www.cnblogs.com/en-heng/p/5041124.html)

这里摘出几个要点
### 定义

> 逆变与协变用来描述类型转换（type transformation）后的继承关系，其定义：如果A、B表示类型，f(⋅)表示类型转换，≤表示继承>关系（比如，A≤B表示A是由B派生出来的子类）；  
> f(⋅)是逆变（contravariant）的，当A≤B时有f(B)≤f(A)成立；  
> f(⋅)是协变（covariant）的，当A≤B时有f(A)≤f(B)成立；  
> f(⋅)是不变（invariant）的，当A≤B时上述两个式子均不成立，即f(A)与f(B)相互之间没有继承关系。

### 理解

>extends确定了泛型的上界，而super确定了泛型的下界

### 区分

> PECS原则：producer-extends, consumer-super.

### 示例

``` java
public static <T> void copy(List<? super T> dest, List<? extends T> src)
...
```

理解了协变确定了泛型的上界，逆变确定了类型的下界，逆变和协变基本就理解清楚了

回到scala的协变和逆变，举一个scala内置单参数函数的trait定义

``` scala
trait Function1 [-T1, +R] extends AnyRef
```

可以理解为函数参数传入是逆变的，函数返回值是协变的

在支持协变和逆变语法的基础上，scala支持类型边界，在泛型编程中通过指定类型的边界，结合逆变协变的语法可以作类型约束

### 通配符_
_可以在模板中匹配类型
``` scala
def count(l: List[_]) = l.size //ignore list element type
```

也可以在Partial application模拟std::bind的语法，从旧的函数中通过部分参数绑定得到新的函数
``` scala
def adder(m: Int, n: Int) = m + n
val add2 = adder(2, _:Int)
```

这和在javascript中经常会使用闭包特性返回部分应用后的新函数是类似的东西

## 对monad的理解

参考：http://hongjiang.info/semigroup-and-monoid/

1. 封闭性（Closure）：对于任意a，b∈G，有a*b∈G 
2. 结合律（Associativity）：对于任意a，b，c∈G，有（a*b）*c=a*（b*c） 
3. 幺元 （Identity）：存在幺元e，使得对于任意a∈G，e*a=a*e=a 
4. 逆元：对于任意a∈G，存在逆元a^-1，使得a^-1*a=a*a^-1=e

**半群（semigroup）：**满足封闭性和结合律

**幺半群（monoid）：** 是半群且有幺元。

**函数(morphism)：**类型之间的映射。

**范畴：**一组类型的集合，为了理解可简单理解为高阶类型。

**函子（functor）：**
一个范畴内的元素可以映射为另一个范畴的元素，且元素之间的关系也可以映射为另一个范畴的关系。对于范畴C1和C2有

1. 将C1中的类型 T 映射为 C2 中的 List[T] :  T => List[T]
2. 将C1中的函数 f 映射为 C2 中的 函数fm :  (A => B) => (List[A] => List[B])

通常带有map方法的类型构造器就是一个函子。

**自函子（endfunctor）：**将范畴映射到自身的函子。

**单子（monad）：**自函子范畴上的一个幺半群。

## 和C++对比
### 优点:
1. 对函数式编程的支持
2. 模式匹配（包括类匹配）能够写出更加简洁的代码
3. 支持高阶函数，C++通过bind和function来模拟其实也问题不大
4. 比C++更加强大的类型推导系统
5. 支持Mixin，通过traits
6. 对类型反射的支持
7. 包比namespace好用，模块化管理
8. 支持注解，目前来说注解还是个好东西
9. 具名参数，C++没有真是遗憾
10. 对于集合的非常强大
11. 对可变参数简单
12. 可以复用jvm上的很多软件包
13. 大数据软件多数是基于scala写的

### 缺点：
1. 过于复杂的语法和操作符
2. 过多的隐式类型转换，隐式转换用起来会很爽
3. 不同版本的jar二进制不兼容
4. 写完全暴露给java调用的代码比较困难，强制scala技术栈
5. 太灵活了，有时候也是缺点
