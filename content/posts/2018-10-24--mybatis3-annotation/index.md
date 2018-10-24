---
title: mybatis3注解调用存储过程
category: "java"
cover: 1.png
---

之前只用过xml版本调用存储过程,在网上也没有找到关于注解版本的demo,对于一些xml的参数,在注解中不知道该如何处理,所以记录下来.

*mapper文件*

```java
public interface CallProceduerMapper {
    @Select("{call GetReportData(#{ProductCode,mode=IN,jdbcType=VARCHAR}, " +
            "            #{Enddate,mode=IN,jdbcType=VARCHAR}, " +
            "            #{ProductStartDate,mode=IN,jdbcType=VARCHAR}, " +
            "            #{conditionId,mode=IN,jdbcType=INTEGER}, " +
            "            #{MerId,mode=IN,jdbcType=VARCHAR}, " +
            "            #{AmountInerval,mode=IN,jdbcType=VARCHAR}, " +
            "            #{GraceDay,mode=IN,jdbcType=INTEGER}," +
            "             #{result,mode=OUT,jdbcType=INTEGER})}")
    @Options(statementType = StatementType.CALLABLE)
    public Integer callReportDate(Map map);
}
```

**注意:**

> 1 存储过程要用 {} 引起来
>
> 2 加注解@Options(statementType = StatementType.CALLABLE) 
>
> 3 返回参数为int 须要用Integer接收

*Service文件*

```java
public int callReportProcedureByParams(Integer cid,ExportRequest request) {
    //设置存储过程需要的参数
    Map procedureMap = new HashMap();
    procedureMap.put("ProductCode",request.getProductCode());
    procedureMap.put("Enddate",endDate);
    procedureMap.put("ProductStartDate",request.getStartDate()==null?"":format.format(request.getStartDate()));
    procedureMap.put("conditionId",cid);
    procedureMap.put("MerId",merId);
    procedureMap.put("AmountInerval",getAmountInerval(amount));
    procedureMap.put("GraceDay",request.getGracePeriod());
    procedureMap.put("result",0);

    logger.info("call report data proceduer params = {}",procedureMap.toString());
    //调用存储过程
    callProceduerMapper.callReportDate(procedureMap);
    //获取返回参数
    Integer presult = (Integer)procedureMap.get("result");

    logger.info("proceduer result ={}",presult);
    if(presult != 1){
        return 0;
    }
    return 1;
}

```

**注意:**

>对于存储过程返回的结果有两种,第一种是存储过程return返回的结果,比如insert之后返回执行结果,这种结果要预设字段接收,如上例. 第二种结果是查询之后返回的结果,比如select之后返回的值,这种在mapper接口中设置对应的javaBean接收即可



|        注解        | 目标      | 相对应的XML       | 描述                                                         |
| :----------------: | --------- | ----------------- | ------------------------------------------------------------ |
|  @CacheNamespace   | 类        | <cache>           | 为给定的命名空间（比如类）配置缓存。属性：implemetation，eviction，flushInterval，size，readWrite，blocking和properties。 |
|     @Property      | N/A       | <property>        | 指定属性值或占位符（可以由mybatis-config.xml中定义的配置属性替换）。属性：name，value。（在MyBatis3.4.2+上可用） |
| @CacheNamespaceRef | 类        | <cacheRef>        | 参照另外一个命名空间的缓存来使用。属性：value，name。如果使用此注释，则应指定value或name属性。value属性指定指定命名空间的java类型（命名空间名称成为指定的java类型的FQCN），对于name属性（此属性自3.4.2起可用）指定命名空间的名称。 |
|  @ConstructorArgs  | 方法      | <constructor>     | 收集一组结果传递给一个劫夺对象的构造方法。属性：value，是形式参数的数组。 |
|        @Arg        | N/A       | `<arg>` `<idArg>` | 单独的构造方法参数，是ConstructorArgs集合的一部分。属性：id，column，javaType，typeHandler。id属性是布尔值，来标识用于比较的属性，和<idArg>XML元素相似。 |
| @TypeDiscriminator | 方法      | <discriminator>   | 一组实例值被用来决定结果映射的表现。属性：column，javaType，jdbcType，typeHandler，cases。cases属性就是实例的数组。 |
|       @Case        | N/A       | <case>            | 单独实例的值和它对应的映射。属性：value，type，results。Results属性是结果数组，因此这个注解和实际的ResultMap很相似，由下面的Results注解指定。 |
|      @Results      | 方法      | <resultMap>       | 结果映射的列表，包含了一个特别结果列如何被映射到属性或字段的详情。属性：value，id。value属性是Result注解的数组。这个id的属性是结果映射的名称。 |
|      @Result       | N/A       | `<result>` `<id>` | 在列和属性或字段之间的单独结果映射。属性：id，column，property，javaType，jdbcType，typeHandler，one，many。id属性是一个布尔值，表示了应该被用于比较（和在XML映射中的<id>相似）的属性。one属性是单独的联系，和<association>相似，而many属性是对集合而言的，和<collection>相似。它们这样命名是为了避免名称冲突。 |
|        @One        | N/A       | <association>     | 复杂类型的单独属性值映射。属性：select，已映射语句（也就是映射器方法）的完全限定名，它可以加载合适类型的实例。注意：联合映射在注解API中是不支持的。这是因为Java注解的限制，不允许循环引用。`fetchType`会覆盖全局的配置参数`lazyLoadingEnabled`。 |
|       @Many        | N/A       | <collection>      | 映射到复杂类型的集合属性。属性：select，已映射语句（也就是映射器方法）的全限定名，它可以加载合适类型的实例的集合，`fetchType`会覆盖全局的配置参数`lazyLoadingEnabled`。注意联合映射在注解API中是不支持的。这是因为Java注解的限制，不允许循环引用 |
|      @MapKey       | 方法      |                   | 复杂类型的集合属性映射。属性：select，是映射语句（也就是映射器方法）的完全限定名，它可以加载合适类型的一组实例。注意：联合映射在Java注解中是不支持的。这是因为Java注解的限制，不允许循环引用。 |
|      @Options      | 方法      | 映射语句的属性    | 这个注解提供访问交换和配置选项的宽广范围，它们通常在映射语句上作为属性出现。而不是将每条语句注解变复杂，Options注解提供连贯清晰的方式来访问它们。属性：useCache=true，flushCache=FlushCachePolicy.DEFAULT，resultSetType=FORWARD_ONLY，statementType=PREPARED，fetchSize=-1，timeout=-1useGeneratedKeys=false，keyProperty=”id”，keyColumn=””，resultSets=””。理解Java注解是很重要的，因为没有办法来指定“null”作为值。因此，一旦你使用了Options注解，语句就受所有默认值的支配。要注意什么样的默认值来避免不期望的行为。 |
|       @Param       | Parameter | N/A               | 如果你的映射器的方法需要多个参数，这个注解可以被应用于映射器的方法参数来给每个参数一个名字。否则，多参数将会以它们的顺序位置来被命名（不包括任何RowBounds参数）比如。#{param1}，#{param2}等，这是默认的。使用@Param（“person”），参数应该被命名为#{person}。 |
|     @SelectKey     | 方法      | <selectKey>       | 该注解复制了`<selectKey>`的功能，用在注解了`@Insert`，`@InsertProvider`，`@Update`or`@UpdateProvider`的方法上。在其他方法上将被忽略。如果你指定了一个`@SelectKey`注解，然后Mybatis将忽略任何生成的key属性通过设置`@Options`，或者配置属性。属性：`statement`是要执行的sql语句的字符串数组，`keyProperty`是需要更新为新值的参数对象属性，`before`可以是`true`或者`false`分别代表sql语句应该在执行insert之前或者之后，`resultType`是`keyProperty`的Java类型，`statementType`是语句的类型，取`Statement`，`PreparedStatement`和`CallableStatement`对应的`STATEMENT`，`PREPARED`或者`CALLABLE`其中一个，默认是`PREPARED`。 |
|     @ResultMap     | 方法      | N/A               | 这个注解给`@Select`或者`@SelectProvider`提供在XML映射中的`<resultMap>`的id。这使得注解的select可以复用那些定义在XML中的ResultMap。如果同一select注解中还存在`@Results`或者`@ConstructorArgs`，那么这两个注解将被此注解覆盖。 |
|    @ResultType     | Method    | N/A               | 当使用结果处理器时启用此注解。这种情况下，返回类型为void，所以Mybatis必须有一种方式决定对象的类型，用于构造每行数据。如果有XML的结果映射，使用`@ResultMap`注解。如果结果类型在XML的`<select>`节点中指定了，就不需要其他的注解了。其他情况下则使用此注解。比如，如果@Select注解在一个方法上将使用结果处理器，返回类型必须是void并且这个注解（或者@ResultMap）是必须的。这个注解将被忽略除非返回类型是void。 |
|       @Flush       | 方法      | N/A               | 如果这个注解使用了，它将调用定义在Mapper接口中的`SqlSession#flushStatements`方法。（Mybatis3.3或者以上） |




