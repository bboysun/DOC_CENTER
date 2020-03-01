## 深入synchronized对象锁

#### 一、对象锁

==关键字synchronized是个对象锁==，意思是取得锁都是对象，而不是把一段代码或方法当做锁，哪个线程先执行带synchronized关键字的方法，哪个线程就先持有该方法所属对象的锁Lock，那么其他线程只能呈等待状态，前提是多个线程访问的是同一个对象。

#### 二、同步与异步

调用synchronized关键字声明的方法一定是排队运行的。另外需要牢牢记住“共享”这两个字，只有共享资源的读写访问才需要同步化（按顺序执行，即串行），如果不是共享资源，那么根本就没有同步化的必要。

常见的场景：

1.A线程先持有Object对象的Lock锁，B线程可以以异步的方式调用Object对象中的非synchronized类型的方法。

2.A线程先持有Object对象的Lock锁，B线程如果在这时调用Object对象中的synchronized类型的方法则需要等待，也就是同步。

#### 三、脏读

如果只是在赋值的时候进行同步，但在取值时有可能出现一些意想不到的意外，这种情况就是脏读（Dirty Read）。发生脏读的情况是在读取实例变量时，此值已经被其他线程更改过了。

举个栗子：

```java
package com.darryl.sun.athena.thread6;

/**
 * @Auther: Darryl
 * @Description: TODO:描述
 * @Date: created in 2019/6/2 15:13
 */

public class PublicVar {

    public String username = "initUsername";
    public String password = "initPassword";

    synchronized public void setValue(String username, String password) throws InterruptedException {
        this.username = username;
        Thread.sleep(5000);
        this.password = password;

        System.out.println("setValue method thread name = " + Thread.currentThread().getName()
        + " username is " + username + " password is " + password);
    }

    public void getValue() {
        System.out.println("getValue method thread name = " + Thread.currentThread().getName()
        + " username is " + username + " password is " + password);
    }

}
```

```java
package com.darryl.sun.athena.thread6;

/**
 * @Auther: Darryl
 * @Description: TODO:描述
 * @Date: created in 2019/6/2 15:19
 */

public class ThreadPubVar extends Thread {
    
    private PublicVar publicVar;

    public ThreadPubVar(PublicVar publicVar) {
        this.publicVar = publicVar;
    }

    @Override
    public void run() {
        super.run();
        try {
            publicVar.setValue("darryl","123456");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

```java
package com.darryl.sun.athena.thread6;

/**
 * @Auther: Darryl
 * @Description: TODO:描述
 * @Date: created in 2019/6/2 15:24
 */

public class TestThread {

    public static void main(String[] args) throws InterruptedException {
        PublicVar publicVar = new PublicVar();
        ThreadPubVar threadPubVar = new ThreadPubVar(publicVar);
        threadPubVar.start();
        // 修改主线线程休眠时间，观察结果如何
        Thread.sleep(200);
        publicVar.getValue();
    }

}
```

运行结果如下：

![1559460613997](C:\Users\Darryl\AppData\Roaming\Typora\typora-user-images\1559460613997.png)

当修改主线程休眠时间为6000时，运行结果如下：

![1559460663592](C:\Users\Darryl\AppData\Roaming\Typora\typora-user-images\1559460663592.png)

我们会发现脏读是因为getValue()方法并不是同步造成的，解决办法自然是在getValue()方法上加synchronized关键字。

#### 四、synchronized锁重入

自己可以再次获取自己的内部锁，比如线程A获取某个对象锁，此时这个对象锁还没有释放，当其再次想要获取这个对象锁的时候还是可以获取的，如果不可锁重入的话，就会造成死锁。

锁重入，也支持在父子类继承的环境中。

#### 五、出现异常，锁自动释放

当一个线程执行的代码出现异常，其所持有的锁会自动释放。

#### 六、同步不具有继承性

同步不可以继承，同步方法被子类覆盖后，同步性无法被继承，需要在子类方法中再次声明才能生效。

#### 七、静态同步synchronized方法

静态同步synchronized方法获取的锁是class锁，class锁可以对类的所有对象实例起作用。