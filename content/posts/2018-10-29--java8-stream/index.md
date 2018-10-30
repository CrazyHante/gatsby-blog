---
title: java8之Stream流
subTitle: java8新特性
category: java
cover: 1.png
---

### 一.Stream通用语法

![stream](./stream.jpg)

- ```
  红色框中的语句是一个Stream的生命开始的地方，负责创建一个Stream实例；
  ```

- ```
  绿色框中的语句是赋予Stream灵魂的地方，把一个Stream转换成另外一个Stream，红框的语句生成的是一个包含
  所有nums变量的Stream，进过绿框的filter方法以后，重新生成了一个过滤掉原nums列表所有null以后的Stream；
  ```

- ```
  蓝色框中的语句是丰收的地方，把Stream的里面包含的内容按照某种算法来汇聚成一个值
  ```

在此我们总结一下使用Stream的基本步骤：

```
1.创建Stream；
2.转换Stream，每次转换原有Stream对象不改变，返回一个新的Stream对象（**可以有多次转换**）；
3.对Stream进行聚合（Reduce）操作，获取想要的结果；
```

### 二.创建Stream

最常用的创建Stream有两种途径：

    1.通过Stream接口的静态工厂方法（注意：Java8里接口可以带静态方法）；
    
    2.通过Collection接口的默认方法（默认方法：Default method，也是Java8中的一个新特性，就是接口中的一个带有实现的方法，后续文章会有介绍）–stream()，把一个Collection对象转换成Stream
##### 2.1 使用Stream静态方法来创建Stream

```tex
1. of方法：有两个overload方法，一个接受变长参数，一个接口单一值

Stream<Integer> integerStream = Stream.of(1, 2, 3, 5);
Stream<String> stringStream = Stream.of("taobao");

2. generator方法：生成一个无限长度的Stream，其元素的生成是通过给定的Supplier（这个接口可以看成一个对象的工厂，每次调用返回一个给定类型的对象）


Stream.generate(new Supplier<Double>() {
    @Override
    public Double get() {
        return Math.random();
    }
});

Stream.generate(() -> Math.random());

Stream.generate(Math::random);

三条语句的作用都是一样的，只是使用了lambda表达式和方法引用的语法来简化代码。每条语句其实都是生成一个无限长度的Stream，其中值是随机的。这个无限长度Stream是懒加载，一般这种无限长度的Stream都会配合Stream的limit()方法来用。

3. iterate方法：也是生成无限长度的Stream，和generator不同的是，其元素的生成是重复对给定的种子值(seed)调用用户指定函数来生成的。其中包含的元素可以认为是：seed，f(seed),f(f(seed))无限循环

Stream.iterate(1, item -> item + 1).limit(10).forEach(System.out::println);
这段代码就是先获取一个无限长度的正整数集合的Stream，然后取出前10个打印。
** 千万记住使用limit方法，不然会无限打印下去 **
```

##### 2.2 通过Collection子类获取Stream
    这个在本文的第一个例子中就展示了从List对象获取其对应的Stream对象，如果查看Java doc就可以发现Collection接口有一个stream方法，所以其所有子类都都可以获取对应的Stream对象。
    
    public interface Collection<E> extends Iterable<E> {
        //其他方法省略
        default Stream<E> stream() {
            return StreamSupport.stream(spliterator(), false);
        }
    }


### 三.转换Stream

##### 3.1 filter 筛选
    filter函数接收一个Lambda表达式作为参数，该表达式返回boolean，在执行过程中，流将元素逐一输送给filter，并筛选出执行结果为true的元素。
    
    List<Person> result = list.stream()
                    .filter(Person::isStudent)
                    .collect(toList());

##### 3.2 distinct 去重
    去掉重复的结果：
    List<Person> result = list.stream()
                    .distinct()
                    .collect(toList())

##### 3.3 limit 截取
    对一个Stream进行截断操作，获取其前N个元素，如果原Stream中包含的元素个数小于N，那就获取其所有的元素；
    List<Person> result = list.stream()
                    .limit(3)
                    .collect(toList());

##### 3.4 skip 跳过
    返回一个丢弃原Stream的前N个元素后剩下元素组成的新Stream，如果原Stream中包含的元素个数小于N，那么返回空Stream；
    List<Person> result = list.stream()
                    .skip(3)
                    .collect(toList());

##### 3.5 map 映射
    对流中的每个元素执行一个函数，使得元素转换成另一种类型输出。流会将每一个元素输送给map函数，并执行map中的Lambda表达式，最后将执行结果存入一个新的流中。
    List<Person> result = list.stream()
                    .map(Person::getName)
                    .collect(toList());

##### 3.6 flagmap 合并多个流
    和map类似，不同的是其每个元素转换得到的是Stream对象，会把子Stream中的元素压缩到父集合中；
    理解为: 将不同的list的流合并为一个list的流
    list.stream()
            .map(line->line.split(" "))
            .flagmap(Arrays::stream)

##### 3.7  peek
    peek: 生成一个包含原Stream的所有元素的新Stream，同时会提供一个消费函数（Consumer实例），新Stream每个元素被消费的时候都会执行给定的消费函数；
    
    Stream.of("one", "two", "three", "four")
     .filter(e -> e.length() > 3)
     .peek(e -> System.out.println("Filtered value: " + e))
     .map(String::toUpperCase)
     .peek(e -> System.out.println("Mapped value: " + e))
     .collect(Collectors.toList());

##### 3.8 anyMatch 是否匹配任一元素
    anyMatch用于判断流中是否存在至少一个元素满足指定的条件，这个判断条件通过Lambda表达式传递给anyMatch，执行结果为boolean类型
    如，判断list中是否有学生：
    boolean result = list.stream()
            .anyMatch(Person::isStudent);
##### 3.9 allMatch 是否匹配所有元素
    allMatch用于判断流中的所有元素是否都满足指定条件，这个判断条件通过Lambda表达式传递给anyMatch，执行结果为boolean类型
    
    如，判断是否所有人都是学生：
    boolean result = list.stream()
            .allMatch(Person::isStudent);

##### 3.10 noneMatch 是否未匹配所有元素
    noneMatch与allMatch恰恰相反，它用于判断流中的所有元素是否都不满足指定条件：
    boolean result = list.stream()
            .noneMatch(Person::isStudent);

##### 3.11 findAny 获取任一元素
    findAny能够从流中随便选一个元素出来，它返回一个Optional类型的元素。
    Optional<Person> person = list.stream()
                                    .findAny();
                                    
    Optional
    Optional是Java8新加入的一个容器，这个容器只存1个或0个元素，它用于防止出现NullpointException，它提供如下方法：
    
    - isPresent() 
    - 判断容器中是否有值。
    - ifPresent(Consume lambda) 
    - 容器若不为空则执行括号中的Lambda表达式。
    - T get() 
    - 获取容器中的元素，若容器为空则抛出NoSuchElement异常。
    - T orElse(T other) 
    - 获取容器中的元素，若容器为空则返回括号中的默认值。

#####  3.12 findFirst 获取第一个元素

    Optional<Person> person = list.stream()
                                    .findFirst();

##### 3.13 reduce 归约
    归约是将集合中的所有元素经过指定运算，折叠成一个元素输出，如：求最值、平均数等，这些操作都是将一个集合的元素折叠成一个元素输出。
       
    reduce函数接收两个参数：
    1.初始值
    2.进行归约操作的Lambda表达式

###### 3.13.1 元素求和：自定义Lambda表达式实现求和
    例：计算所有人的年龄总和
    int age = list.stream().reduce(0, (person1,person2)->person1.getAge()+person2.getAge());

###### 3.13.2 元素求和：使用Integer.sum函数求和
    int age = list.stream().reduce(0, Integer::sum);

##### 3.14 数值流的使用
    采用reduce进行数值操作会涉及到基本数值类型和引用数值类型之间的装箱、拆箱操作，因此效率较低。 
    当流操作为纯数值操作时，使用数值流能获得较高的效率。
    
    StreamAPI提供了三种数值流:
    1.IntStream
    2.DoubleStream
    3.LongStream
    将普通流转换成数值流的三种方法:
    1.mapToInt
    2.mapToDouble
    3.mapToLong
    
    如，将Person中的age转换成数值流：
    IntStream stream = list.stream()
                            .mapToInt(Person::getAge);
                            
    每种数值流都提供了数值计算函数，如max、min、sum等。 
    如，找出最大的年龄：
    OptionalInt maxAge = list.stream()
                                .mapToInt(Person::getAge)
                                .max();
    此外，mapToInt、mapToDouble、mapToLong进行数值操作后的返回结果分别为：OptionalInt、OptionalDouble、OptionalLong