# 【Dart】站在Java的时间简单理解Dart中的extend、implement与with

### 前言

Flutter使用了Dart作为其编程语言，得益于Dart的易于上手，使得具有一定其他语言基础（如Java等）的人上手起来比较容易。但是区别于Java，Dart不止拥有着`extend`, `implement` 这2种修饰符，Dart还有 `with` 这个Java没有的修饰符，而且Dart中的 `extend` 和 `implement` 于Java中的用法也有区别。这里做一个简单的记录，记录下这3个修饰符在Dart中的用法

### 特点

1. **implement**

   `implement` 在Java中为描述一个类导入了某个接口 ( `interface` ) 并需要实现这个接口所定义的方法。比如下面这个例子

   ```java
       interface A {
           
           void method();
           
       }
       
       class B implements A{
   
           @Override
           public void method() {
               // TODO...
           }
           
       }
   ```

   例子中，接口A定义有`method()`这一个方法，而类B `implement` 了A，因此B也需要实现A所定义的所有接口方法，同时允许多个`implement`，如有多个`implement`，那么则需要实现所有接口所定义的方法。

   而在Dart中也是类似的概念，只不过区别在于 Dart 没有 `interface` 这样的关键字，在Dart中每一个类都是一个隐式的接口，这个接口包括类里面定义的所有成员变量、定义的方法。

   以上面的例子为例，如果我们把他转换成Dart表示，将会是这样子的

   ```dart
   class A{
     
     void method(){
       
     }
     
   }
   
   class B implements A{
     
     @override
     void method() {
       // TODO...
     }
     
   }
   ```

   当然，如果A有成员变量的话，B也需要有相同的成员变量

   ```dart
   class A{
     int a = 20;
   
     void method(){
   
     }
   
   }
   
   class B implements A{
     int a = 30;
   
     @override
     void method() {
       // TODO...
     }
   
   }
   ```

   因此Dart中关于`implement`，按照JAVA的思想可以简单理解为

   + 每一个类其实就是一个隐式的 interface
   + implement 不仅仅需要实现其方法，还需要实现其成员变量
   + implement 不会继承其实现
   + 允许 implement 多个，但是同时也需要实现所有定义的接口方法以及其成员变量

   

2. **extend**

   Dart中的extend与Java中的extend是非常相似的

   + Dart与Java中的extend只能是单继承
   + Dart与Java中的构造函数不能继承
   + 子类重写父类方法需要使用`@override`
   + 子类调用父类的实现，需要使用`super`

   但是这其中也有些地方不尽相同，由于Dart没有Java那么多的访问权限修饰符（其实也不多...），Flutter中的子类是可以访问父类的所有变量与方法，因为Dart并没有共有与私有的的区别

   如下面的例子

   ```java
       class A {
           private int a;
           protected int b;
           int c;
           public int d;
           
           private void methodA(){  }
           protected void methodB(){  }
           void methodC(){  }
           public void methodD(){  }
           
       }
   
       class B extends A{
           
   
           @Override
           protected void methodB() {
               super.methodB();
               b++;
           }
   
           @Override
           void methodC() {
               super.methodC();
               c++;
           }
   
           @Override
           public void methodD() {
               super.methodD();
               d++;
           }
           
       }
   ```

   对应Dart如下

   ````dart
   class A{
     int a = 20;
     int _b = 50;
   
     void methodA(){ }
     
     void methodB(){ }
   
   }
   
   class B extends A{
     @override
     void methodA() {
       super.methodA();
       a++;
     }
     
     @override
     void methodB() {
       super.methodB();
       _b++;
     }
   
   }
   ````

   

3. **with ( 混入 )**

   `with` 区别于JAVA为Dart独有。是Dart的重要特性( mixins )。

   mixins 的中文意思为混入，其定义如下

   >  Mixins are a way of reusing a class’s code in multiple class hierarchies. 
   >
   >  Mixins是一种在多个类层次结构中复用类代码的方法 。

   关于更多Dart Mixin的介绍可以看看这篇翻译

   [【译】Dart | 什么是Mixin]( https://www.jianshu.com/p/a578bd2c42aa )

   通过Mixin可以做到使得一个类可以混入多个类，并且拥有并重写他们的成员变量和方法，比如说下面的例子

   ```dart
   class A{
     int a = 1;
   
     void methodA(){
       print('A(a = $a)');
     }
     
   }
   
   class B{
     int b = 2;
     
     void methodB(){
       print('B(b = $b)');
     }
     
   }
   
   class C{
     int c = 3;
     
     void methodC(){
       print('C(c = $c)');
     }
   }
   
   class M with A,B,C{
     int a = 11;
     int b = 22;
     
     @override
     void methodC() {
       print('override $c');
     }
     
   }
   ```

   可以看到，M 混入 A，B，C 后，不仅仅能使用，还能重写A,B,C 的中的方法与成员变量

   如果说这其中混入的方法里面存在相同的方法名或者成员变量名，则会按照混入顺序来决定调用的是那个方法

   比如这个例子

   ```dart
   class A{
   
     void method(){}
   
   }
   
   class B{
   
     void method(){}
   
   }
   
   class C{
   
     void method(){}
   
   }
   
   class D extends A with B,C{
   
   }
   
   D().method();
   ```

   最后执行的时候，将执行的是 C 的 `method()` 方法

### 参考资料

[Vadaski - 【译】Dart | 什么是Mixin]( https://www.jianshu.com/p/a578bd2c42aa )

[小德_Kurtl -  Flutter Dart mixins 探究]( https://juejin.im/post/5c44382d51882523f0261bb5 )

[小德_Kurtl  - Flutter Dart语法(1):extends 、 implements 、 with的用法与区别]( https://juejin.im/post/5c4881dae51d45098e4d96cf )