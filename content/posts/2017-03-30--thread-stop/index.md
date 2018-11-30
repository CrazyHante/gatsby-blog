---
title: 线程停止的方法
subTitle: thread
category: java
cover: java.png
---

1.采用标志位的方式终止

```java
public class ThreadFlag extends Thread  { 
    public volatile boolean exit = false; 

    public void run()  { 
        while (!exit){
            //doSomething
        } 
    } 
    public static void main(String[] args) throws Exceptio { 
        ThreadFlag thread = new ThreadFlag(); 
        thread.start(); 
        sleep(5000); // 主线程延迟5秒 
        thread.exit = true;  // 终止线程thread 
        thread.join(); 
        System.out.println("线程退出!"); 
    } 
}
```

在上面代码中定义了一个退出标志exit，当exit为true时，while循环退出，exit的默认值为false.在定义exit时，使用了一个Java关键字volatile，这个关键字的目的是使exit同步，也就是说在同一时刻只能由一个线程来修改exit的值

2.使用interrupt方法终止阻塞线程 如果线程处于阻塞状态,比如sleep() 那此时就算修改exit标志位也不会停止线程

```java
public class ThreadFlag extends Thread  { 
    public volatile boolean exit = false; 

    public void run()  { 
        while (!exit){
            try{
                sleep(1000);
            }catch(InterruptedException e){
                System.out.println("week up from blcok...");
                stop = true; // 在异常处理代码中修改共享变量的状态
            }
            
        } 
    } 
    public static void main(String[] args) throws Exceptio { 
        ThreadFlag thread = new ThreadFlag(); 
        thread.start(); 
        sleep(5000); // 主线程延迟5秒 
        thread.exit = true;  // 终止线程thread 
        thread.interrupt(); // 阻塞时退出阻塞状态
        thread.join(); 
        System.out.println("线程退出!"); 
    } 
}
```

注意：在Thread类中有两个方法可以判断线程是否通过interrupt方法被终止。一个是静态的方法interrupted（），一个是非静态的方法isInterrupted（），这两个方法的区别是interrupted用来判断当前线是否被中断，而isInterrupted可以用来判断其他线程是否被中断。