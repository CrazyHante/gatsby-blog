---
title: JVM类加载器 ClassLoader(二)
subTitle: jvm整理
category: java
cover: java.png
---

ClassLoader的作用

(1)加载class文件进入JVM 

(2)审查每个类应该由谁加载，采用双亲委托机制 

(3)将class字节码重新解析成JVM要求的对象格式



JVM中提供的三个ClassLoader类，下图示层次 

![img](/clipboard.png)

关系:

1. Bootstrp loader 是在Java虚拟机启动后初始化的。它仅仅是一个类的加载工具，既没有父加载器也没有子加载器。 
2. Bootstrp loader 负责加载 ExtClassLoader,并且将 ExtClassLoade r的父加载器设置为 Bootstrp loader。
3. Bootstrp loader 加载完 ExtClassLoader 后，就会加载 AppClassLoader,并且将 AppClassLoader 的父加载器指定为 ExtClassLoader。





JVM加载Class文件到内存的两种方式

(1)隐式加载：在代码中不通过调用ClassLoader来加载需要的的类，而是通过JVM自动加载需要的类到内存方式。如我们类继承或类中引用其他类的时候，JVM在解析当前这个类的时候发现引用类不在内存中就会去自动的将这些类加载进入到内存。 

(2)显示加载：在代码中通过调用ClassLoader类来加载类。如this.getClass().getClassLoader().loadClass()或者Class.forName()或者我们自己实现的findClass()方法。



双亲委托模型

​    Java中ClassLoader的加载采用了双亲委托机制，采用双亲委托机制加载类的时候采用如下的几个步骤：

1. 当前ClassLoader首先从自己已经加载的类中查询是否此类已经加载，如果已经加载则直接返回原来已经加载的类。
2. 当前classLoader的缓存中没有找到被加载的类的时候，委托父类加载器去加载，父类加载器采用同样的策略，首先查看自己的缓存，然后委托父类的父类去加载，一直到bootstrp ClassLoader.
3. 当所有的父类加载器都没有加载的时候，再由当前的类加载器加载，并将其放入它自己的缓存中，以便下次有加载请求的时候直接返回。

ClassLoader的源码

![img](/clipboard1.png)



采用双亲委托模式可以防止重复加载, 同时也可以防止用户自定义的类加载器代替java的加载器



类的绳命周期

![img](/clipboard2.png)

类加载过程

![img](/clipboard3.png)





(1)第一阶段：找到class文件并把这个文件包含的字节码加载到内存 

(2)第二阶段：字节码验证，class类数据结构分析以及相应内存分配，符号表的链接。 

(3)第三阶段：类中静态属性初始化赋值以及静态代码块的执行



第一阶段:加载字节码到内存 (加载class文件到内存)

(1)通过一个类的全限定名来获取定义此类的二进制字节流 

(2)将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构 

(3)在java堆中生成一个代表这个类的java.lang.Class对象，作为方法区这些数据的访问入口



第二阶段 :链接（Linking）是检验类或接口并准备类型和父类接口的过程。链接过程包含三步：验证（Verifying）、准备（Preparing）、部分解析（Optionally resolving）。



验证：这是类装载中最复杂的过程，并且花费的时间也是最长的。任务是确保导入类型的准确性，验证阶段做的检查，运行时不需要再做，虽然减慢加了载速度，但是避免了多次检查。

准备：分配一个结构用来存储类信息，这个结构中包含了类中定义的成员变量，方法和接口的信息。

解析：可选阶段，把这个类的常量池中的所有的符号引用改变成直接引用。如果不执行，符号解析要等到字节码指令使用这个引用时才会进行



第三阶段:初始化（Initialization）把类中的变量初始化成合适的值。执行静态初始化程序，把静态变量初始化成指定的值,执行静态代码块.



