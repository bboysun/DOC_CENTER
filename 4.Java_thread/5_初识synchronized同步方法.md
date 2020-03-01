## 初识synchronized同步方法

一、方法内的变量是线程安全的。

因为方法内部的变量是私有的，并且会随着方法的销毁而被销毁。可见方法中的变量不存在非线程安全问题，永远都是线程安全的。这是方法内部变量是私有的特性造成的。

二、实例变量（一个类的全局变量）非线程安全

如果多个线程共同访问一个对象中的实例变量，则有可能出现非线程安全问题。话不多说，举个栗子：

```java
package com.darryl.sun.athena.thread5;

/**
 * @Auther: Darryl
 * @Description: TODO:描述
 * @Date: created in 2019/6/2 14:44
 */

public class HasSelfPrivateNum {

    private int num = 0;

    public void addI(String username) {
        if ("a".equals(username)) {
            num = 100;
            System.out.println("a set over~");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        } else {
            num = 200;
            System.out.println("b set over~");
        }
        System.out.println(username + " num = " + num);
    }

}
```

```java
package com.darryl.sun.athena.thread5;

/**
 * @Auther: Darryl
 * @Description: TODO:描述
 * @Date: created in 2019/6/2 14:47
 */

public class ThreadA extends Thread {

    private HasSelfPrivateNum hasSelfPrivateNum;

    public ThreadA(HasSelfPrivateNum hasSelfPrivateNum) {
        super();
        this.hasSelfPrivateNum = hasSelfPrivateNum;
    }

    @Override
    public void run() {
        super.run();
        hasSelfPrivateNum.addI("a");
    }
}
```

```java
package com.darryl.sun.athena.thread5;

/**
 * @Auther: Darryl
 * @Description: TODO:描述
 * @Date: created in 2019/6/2 14:49
 */

public class ThreadB extends Thread {

    private HasSelfPrivateNum hasSelfPrivateNum;

    public ThreadB(HasSelfPrivateNum hasSelfPrivateNum) {
        this.hasSelfPrivateNum = hasSelfPrivateNum;
    }

    @Override
    public void run() {
        super.run();
        hasSelfPrivateNum.addI("b");
    }
}
```

```java
package com.darryl.sun.athena.thread5;

/**
 * @Auther: Darryl
 * @Description: TODO:描述
 * @Date: created in 2019/6/2 14:50
 */

public class GoGoGo {

    public static void main(String[] args) {
        HasSelfPrivateNum hasSelfPrivateNum = new HasSelfPrivateNum();

        ThreadA threadA = new ThreadA(hasSelfPrivateNum);
        threadA.start();
        ThreadB threadB = new ThreadB(hasSelfPrivateNum);
        threadB.start();

    }

}
```

运行结果如下：

![1559458448905](C:\Users\Darryl\AppData\Roaming\Typora\typora-user-images\1559458448905.png)

我们会发现此时就出现了非线程安全问题。

那么要如何处理呢？

最简单的办法就是在addI()方法前加关键字synchronized即可。更改后的代码如下：

```java
package com.darryl.sun.athena.thread5;

/**
 * @Auther: Darryl
 * @Description: TODO:描述
 * @Date: created in 2019/6/2 14:44
 */

public class HasSelfPrivateNum {

    private int num = 0;

    synchronized public void addI(String username) {
        if ("a".equals(username)) {
            num = 100;
            System.out.println("a set over~");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        } else {
            num = 200;
            System.out.println("b set over~");
        }
        System.out.println(username + " num = " + num);
    }

}
```

运行结果如下：

![1559458607330](C:\Users\Darryl\AppData\Roaming\Typora\typora-user-images\1559458607330.png)

见识到synchronized的厉害了吧~