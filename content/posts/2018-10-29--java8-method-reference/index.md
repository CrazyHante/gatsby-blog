---
title: Java8之方法引用
subTitle: java8新特性
category: java
cover: 1.png
---

####    一、概述

```tex
在学习lambda表达式之后，我们通常使用lambda表达式来创建匿名方法。
然而，有时候我们仅仅是调用了一个已存在的方法。如下：

Arrays.sort(stringsArray,(s1,s2)->s1.compareToIgnoreCase(s2));

在Java8中，我们可以直接通过方法引用来简写lambda表达式中已经存在的方法。

Arrays.sort(stringsArray, String::compareToIgnoreCase);

这种特性就叫做方法引用(Method Reference)。
```

####    二、什么是方法引用

```tex
 方法引用是用来直接访问类或者实例的已经存在的方法或者构造方法。方法引用提供了一种引用而不执行方法的方式，它需要由兼容的函数式接口构成的目标类型上下文。计算时，方法引用会创建函数式接口的一个实例。

当Lambda表达式中只是执行一个方法调用时，不用Lambda表达式，直接通过方法引用的形式可读性更高一些。方法引用是一种更简洁易懂的Lambda表达式。

注意方法引用是一个Lambda表达式，其中方法引用的操作符是双冒号"::"。
```


####    三、四种方法引用类型

    方法引用的标准形式是：类名::方法名。（注意：只需要写方法名，不需要写括号）

==**有以下四种形式的方法引用**：==
​    
| 类型                             | 示例                                 |
| -------------------------------- | ------------------------------------ |
| 引用静态方法                     | ContainingClass::staticMethodName    |
| 引用某个对象的实例方法           | containingObject::instanceMethodName |
| 引用某个类型的任意对象的实例方法 | ContainingType::methodName           |
| 引用构造方法                     | ClassName::new                       |


####  四.方法引用例子
```java
public class Person {
    public enum Sex{
        MALE,FEMALE
    }
 
    String name;
    LocalDate birthday;
    Sex gender;
    String emailAddress;
 
    public String getEmailAddress() {
        return emailAddress;
    }
 
    public Sex getGender() {
        return gender;
    }
 
    public LocalDate getBirthday() {
        return birthday;
    }
 
    public String getName() {
        return name;
    }
 
    public static int compareByAge(Person a,Person b){
        return a.birthday.compareTo(b.birthday);
    }
 
}
```
##### 1.引用静态方法

```java
    Person [] persons = new Person[10];
    //使用匿名类
    Arrays.sort(persons, new Comparator<Person>() {
                @Override
                public int compare(Person o1, Person o2) {
                    return o1.birthday.compareTo(o2.birthday);
                }
     });
    
    //使用lambda表达式
    Arrays.sort(persons, (o1, o2) -> o1.birthday.compareTo(o2.birthday));
    
    //使用lambda表达式和类的静态方法
    Arrays.sort(persons, (o1, o2) -> Person.compareByAge(o1,o2));
    
    //使用方法引用
    //引用的是类的静态方法
    Arrays.sort(persons, Person::compareByAge);
```

##### 2.引用对象实例方法
```java
理解为:创建对象后,再调用对象里面的方法

class ComparisonProvider{
            public int compareByName(Person a,Person b){
                return a.getName().compareTo(b.getName());
            }
 
            public int compareByAge(Person a,Person b){
                return a.getBirthday().compareTo(b.getBirthday());
            }
        }
 
ComparisonProvider provider = new ComparisonProvider();
 
//使用lambda表达式
//对象的实例方法
Arrays.sort(persons,(a,b)->provider.compareByAge(a,b));
 
//使用方法引用
//引用的是对象的实例方法
Arrays.sort(persons, provider::compareByAge);
```

##### 3.引用类型对象的实例方法

```java
理解为:String,Integer这种java内置的引用对象的内置方法
String[] stringsArray = {"Hello","World"};
 
//使用lambda表达式和类型对象的实例方法
Arrays.sort(stringsArray,(s1,s2)->s1.compareToIgnoreCase(s2));
 
//使用方法引用
//引用的是类型对象的实例方法
Arrays.sort(stringsArray, String::compareToIgnoreCase);
```

##### 4.引用构造方法
```java
public static <T, SOURCE extends Collection<T>, DEST extends Collection<T>>
    DEST transferElements(SOURCE sourceColletions, Supplier<DEST> colltionFactory) {
        DEST result = colltionFactory.get();
        for (T t : sourceColletions) {
            result.add(t);
        }
        return result;
    }
...
  
final List<Person> personList = Arrays.asList(persons);
 
//使用lambda表达式
Set<Person> personSet = transferElements(personList,()-> new HashSet<>());
 
//使用方法引用
//引用的是构造方法
Set<Person> personSet2 = transferElements(personList, HashSet::new); 
```