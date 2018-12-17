# 【JAVA】关于使用finally导致异常丢失

## 简单回顾

相信大家有接触过JAVA的都不可避免的会涉及到异常处理把，异常处理旨在提供一种更友好方法来处理一些影城程序运行中的一些意料之外的事情。比如说空指针引用异常`NullPointerException`、数组下标越界异常`IndexOutOfBoundException`、内存不足异常`OutOfMemoryException`和各种各样我们自己定义的异常。

在Java的异常中都是继承自`Throwable`类的，可以分为2类异常，一种需要我们显性捕获的异常，比如说在操作套接字Socket的时候经常会遇到的`IOException`，这类异常需要我们显性的使用`try...catch`语句块进行捕获处理；还有一种就是系统异常，这类异常Java虚拟机会帮我们捕获，比如是内存不足、空指针引用、除以0等等。

在使用`try...catch`语句块时，我们可以对捕获到的异常进行进一步的处理，以便恢复应用程序的执行或者把异常上报到上一层中去处理。但是使用`try...catch`语句块进行处理时我们可能会因为一时疏忽导致遗漏了某个处理方法（虽然说，你可以直接捕获Exception类，但是，不建议这样干），或者是异常捕获后需要同样的处理让应用程序恢复到正常的状态。那么`try...catch...finally`将会帮上大忙。

`finally`语句块里面的语句将能确保执行，无论是你直接在`catch`中直接`return`了整个函数，`finally`中的语句也能执行，这样在我们需要做统一处理恢复程序正常运行提供极大的便利。

## 问题描述

由此可见`finally`语句块的确能给我们提供不小的便利。但是，在使用`finally`的时候也要小心使用，否则可能会出现异常丢失的问题。我们可以看看下面的2个例子

## Example #1

```java
class ExceptionOne extends Exception{}
class ExceptionTwo extends Exception{}

public class Main {
    
    private void one() throws ExceptionOne{
        throw new ExceptionOne();
    }
    
    private void two() throws ExceptionTwo{
        throw new ExceptionTwo();
    }

    public static void main(String[] args){
        try {
            Main main = new Main();
            try {
                main.one();
            }finally {
                main.two();
            }
        }catch (Exception e){
            e.printStackTrace();
        }
    }

}
```

执行上面的代码，我们得到的输出如下

```
ExceptionTwo
	at Main.two(Main.java:9)
	at Main.main(Main.java:18)
	
Process finished with exit code 0
```

发现问题了吧，这里只有`ExceptionTwo`的异常信息，而在`ExceptionOne`的信息却因为在`finally`语句块中重新抛出异常从而丢失了`ExceptionOne`的异常信息。应付这种问题，我个人的开发是如果需要在`finally`语句块中需要处理的函数可能会抛出异常的话，我们需要在`finally`语句块中处理的的时候也同样需要捕获异常信息。当然，这只是个人肤浅的认识，欢迎各位交流

## Example #2

```java
public class Main {

    public static void main(String[] args){
        try{
            throw new NullPointerException();
        }finally {
            return;
        }
    }

}
```

执行上面的代码，我们将会得到输出

```
Process finished with exit code 0
```

可以看到，`NullPointerException`就这样悄无声息的消失了。导致这个问题的原因我也还没搞懂，把其中`return;`注释掉的话，异常是能正确抛出来的。有哪位大神如果知道的话不妨赐教一下小弟我，小弟在此谢过啦