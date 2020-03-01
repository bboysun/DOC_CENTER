## volatile关键字

关键字volatile的主要作用就是使变量在多个线程间可见。

先看看他能帮我们解决哪些问题吧~

一、解决同步死循环

话不多说，举个栗子：

```java
package com.darryl.sun.athena.thread9;

/**
 * @Auther: Darryl
 * @Description: TODO:描述
 * @Date: created in 2019/6/9 14:01
 */

public class PrintString implements Runnable {

    private boolean isContinue = true;

    public void setIsContinue(boolean isContinue) {
        this.isContinue = isContinue;
    }

    public void printStringMethod() {
        while (isContinue == true) {
            System.out.println("run printStringMethod threadName is "
                    + Thread.currentThread().getName());
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    @Override
    public void run() {
        this.printStringMethod();
    }
}
```

```java
package com.darryl.sun.athena.thread9;

/**
 * @Auther: Darryl
 * @Description: TODO:描述
 * @Date: created in 2019/6/9 14:06
 */

public class Go2Run {

    public static void main(String[] args) {
        PrintString printString = new PrintString();
        new Thread(printString).start();
        System.out.println("I want to stop it after 3sec, stopThread name is "
                + Thread.currentThread().getName());
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        printString.setIsContinue(false);
    }
}

```

运行结果如下：

![1560061087317](C:\Users\Darryl\AppData\Roaming\Typora\typora-user-images\1560061087317.png)

但上面的栗子，运行在-server服务器模式中的64bit的JVM上时，会出现死循环。解决的办法就是使用volatile关键字。

修改上面的代码如下：

```java
volatile private boolean isContinue = true;
```

关键字volatile的作用就是强制从公共堆栈中获取变量值，而不是从线程私有数据栈中获取变量值。

上面的栗子在server端之所以会出现死锁现象，主要原因就是私有堆栈中的值和公共堆栈中的值不同步造成的。

原本的线程都是从私有堆栈中对成员变量A取值进行逻辑计算，加上volatile关键字以后的成员变量A，线程对A取值则是直接从公共内存中读取变量值。

对比synchronized，我们会发现synchronized是保证线程原子性，而volatile是保证线程可见性，java同步机制都是围绕这两个方面展开来确保线程安全的。

==:notebook: 冷知识：原子类型 AtomicInteger原子类型的integer变量类型。这种类型有很多比如AtomicLong等等==

重点来了：

关键字Synchronized可以保证在同一时刻，只有一个线程可以执行某一个方法或者某一个代码块。它包含两个特征：互斥性和可见性。同步synchronized不仅可以解决一个线程看到对象处于不一致的状态，还可以保证进入同步方法或者同步代码块的每个线程，都看到由同一个锁保护之前所有的修改效果。

jmm java的内存模型，对于同步synchronized关键字有两条规定：

1. 线程解锁时，必须把共享变量的最新值刷新到主内存中。
2. 线程加锁时，将清空工作内存中共享变量的值，从而使用共享变量时，需要从主内存中重新获取最新的值。
3. 通过这种方法来实现synchronized的可见性。

学习多线程并发，着重“外练互斥，内修可见”。