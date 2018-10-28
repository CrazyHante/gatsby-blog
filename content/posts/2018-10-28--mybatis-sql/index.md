---
title: mybatis3注解之动态sql
subTitle: mybatis3
category: java
cover: 1.png
---


 有时候，我们需要在输入的标准下，创建动态的查的语言。MyBatis提供了多个注解如：@InsertProvider,@UpdateProvider,@DeleteProvider和@SelectProvider，这些都是建立动态语言和让MyBatis执行这些语言。
```java
public interface TutorMapper {
    @InsertProvider(type = TutorDynaSqlProvider.class, method = "insertTutor")
    @Options(useGeneratedKeys = true, keyProperty = "tutorId")
    int insertTutor(Tutor tutor);
}
```

```java
public class TutorDynaSqlProvider {
    public String insertTutor(final Tutor tutor) {
        return new SQL() {
            {
                INSERT_INTO("TUTORS");
                if (tutor.getName() != null) {
                    VALUES("NAME", "#{name}");
                }
                if (tutor.getEmail() != null) {
                    VALUES("EMAIL", "#{email}");
                }
            }
        }.toString();
    }
}

type指向动态sql的工厂类,method为具体的方法
使用@Options注解的userGeneratedKeys 和keyProperty属性让数据库产生auto_increment（自增长）列的值，然后将生成的值设置到输入参数对象的属性中
有一些数据库如Oracle，并不支持AUTO_INCREMENT列属性，它使用序列（SEQUENCE）来产生主键的值。

我们可以使用@SelectKey注解来为任意SQL语句来指定主键值，作为主键列的值。

例如:
@Insert("INSERT INTO STUDENTS(STUD_ID,NAME,EMAIL,ADDR_ID, PHONE)   
VALUES(#{studId},#{name},#{email},#{address.addrId},#{phone})")  
@SelectKey(statement="SELECT STUD_ID_SEQ.NEXTVAL FROM DUAL",   
keyProperty="studId", resultType=int.class, before=true)  
int insertStudent(Student student);  
```


---------------

> 参考:
>
> - http://blog.csdn.net/owen_william/article/details/51815506
> - https://www.cnblogs.com/zhangminghui/p/4903351.html
> - http://blog.csdn.net/u013214151/article/details/52211614

