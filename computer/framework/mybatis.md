# Mybatis





## 基本内容

### 传统jdbc出现的问题

主要问题是==硬编码sql语句== 我们的解决方案就是将sql语句转移到配置文件当中... mybatis应运而生

1. 解决sql语句中参数位置 参数名称的硬编码
2. 解决结果集到java对象的硬编码



### Mybatis是什么?

是持久层的框架 不是ORM框架... 我们的主要精力放在了sql上



### 基础代码

核心配置文件是`SqlMapConfig.xml` 可以配置数据源/事务 整合之后会被Spring接管

`Mapper.xml` 就是sql语句到函数的映射

核心API是`SqlSessionFactory` `SqlSession(Executor)`

Executor

MappedStatement





## SqlMapConfig.xml

配置文件格式:

properties => settings => typeAlises => typeHandlers => environments => mappers



==<properties>==用来加载其他的配置文件





## MBG(逆向工程)

MyBatis Genertor 简称 `MGB`, 是一个专门为Mybatis框架定制的代码生成器, 可以快速的生成`Bean`/`接口`/`配置`, 支持基本的`增删改查`以及`QBC`风格的查询条件, 但是复杂的SQL语句以及存储过程等还是需要我们手动编写, 我们仅仅需要从数据库中创建出表, 然后就可以进行生成了. 我们可以参考如下的网站

* [Mybatis官方文档](http://www.mybatis.org/generator/quickstart.html): 可以提供配置文件模板, Java运行类的代码模板等...
* [Mybatis的GitHub地址](https://github.com/mybatis/generator): 可以提供包的下载等...



### 思路

* MGB最终都需要`运行一个"程序"`来生成我们需要的`Bean+接口+配置`, 这个程序可能是:
  * `Maven插件`: 利用mvn命令
  * `普通Java类`: 利用main()方法或者JUnit
  * `Java的可运行jar包`: 利用java -jar的方式

* 无论使用哪一种`"程序"`, 我们都应该向程序提供如下的信息:
  * `JDBC驱动包`: 只有程序有了JDBC驱动 才能**连接数据库**进行逆向工程
  * `MBG配置文件的位置`: 只有知道了配置文件的位置 **"程序"**才能知道:
    * 数据源的配置信息... 否则无法连接到数据库 也不知道连接哪个数据库
    * 利用数据库的哪张表进行生成....
    * 生成的JavaBean和接口配置文件等放在哪里?
    * 生成的资源是否都进行注释处理?
    * ......

> 其实我们的MGB配置文件中就可以去配置`JDBC驱动包`的详细位置, 这样我们就不用向程序去提供了



### 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <!-- 配置数据源的驱动包所在位置 如果不配 请在程序中提供 -->
    <!--<classPathEntry location="/Program Files/IBM/SQLLIB/java/db2java.zip" />-->

    <context id="DB2Tables" targetRuntime="MyBatis3">

        <!-- 如果配置为true 会省去注释信息 -->
        <commentGenerator>
            <property name="suppressAllComments" value="true"/>
            <property name="suppressDate" value="true" />
        </commentGenerator>

        <!-- 指定数据库的连接信息 -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/test"
                        userId="root"
                        password="root">
        </jdbcConnection>
        
        <!-- 指定JavaBean的生成策略 -->
        <javaModelGenerator targetPackage="me.faene.pojo" targetProject="./src/main/java">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>

        <!-- 指定Sql映射文件的生成策略 -->
        <sqlMapGenerator targetPackage="me.faene.mapper"  targetProject="./src/main/java">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>

        <!-- 指定Mapper接口的生成策略 -->
        <javaClientGenerator type="XMLMAPPER" targetPackage="me.faene.mapper"  targetProject="./src/main/java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>

        <!-- 通过数据库中的那些表进行生成 -->
        <table tableName="emp" domainObjectName="Emp"></table>
        <table tableName="dept" domainObjectName="Dept"></table>

    </context>
</generatorConfiguration>
```



### Maven插件方式

如果使用Maven插件的方式 我们需要在我们的项目中导入Maven插件:

```xml
<plugin>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-maven-plugin</artifactId>
    <version>1.3.7</version>
    <configuration>
        <!-- MBG配置文件的路径 相对于项目路径下 -->
        <configurationFile>generatorConfig.xml</configurationFile>
        <!-- 如果连续生成多次 则会覆盖之前的生成内容 -->
        <overwrite>true</overwrite>
    </configuration>
    <!-- 逆向工程需要知道我们JDBC驱动的所在位置 -->
    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.27</version>
        </dependency>
    </dependencies>
</plugin>
```

我们在Maven插件中依赖了`JDBC驱动`, 也指定了`配置文件的位置` 然后直接通过如下Maven命令运行插件即可:

```txt
mybatis-generator:generate
```



### Java类

使用Java类的方式首先应该导入所要依赖的jar包, 我们必须要提供`JDBC驱动` 除非你再配置文件中配置好了

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.27</version>
</dependency>
<dependency>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-maven-plugin</artifactId>
    <version>1.3.7</version>
</dependency>
```

既然通过Java类的方式, 我们首先创建一个基本的Java类, 提供Main方法或者Junit测试方法:

```java
public static void main(String[] args) {
    List<String> warnings = new ArrayList<String>();
    // 设置是否覆盖之前生成的内容
    boolean overwrite = true;
    // 指定配置文件的位置 File类在项目中相对路径就是项目下
    File configFile = new File("generatorConfig.xml");
    ConfigurationParser cp = new ConfigurationParser(warnings);
    Configuration config = cp.parseConfiguration(configFile);
    DefaultShellCallback callback = new DefaultShellCallback(overwrite);
    MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
    myBatisGenerator.generate(null);
}
```





### Q:重新生成代码时的SQL映射文件覆盖

如果我们连续生成多次... 即便配置好了`overwrite=true` 但是对于`SQL映射文件` 并不会真的去覆盖, 而是采用了追加的方式... 这样就会导致我们的代码出错... 所以说:

**每次执行逆向工程生成代码之前 一定要删除以前生成好的内容......**



所以我们的建议就是 不要将`SQL映射文件`放在与`Mapper接口`一样的文件夹中了... 否则... 太难处理

`/rescoues/mapper`: 用作自己书写

`/resouces/mapper-generator`: 用作生成的...



### 使用方式

我们使用逆向工程生成的Mapper类已经为我们提供了如下的方法:

* `countByExample(TExample):int`

* `selectByPrimaryKey(ID id):List<T>` `selectByExample(TExample):List<T>`
* `deleteByPrimaryKey(ID id):int` `deleteByExample(XxxExample):int`
* `insert(T):int` `insertSelective(T):int`
* `updateByPrimaryKey(T t):int` `updateByExample(T record, TExample example):int`
* `updateByPrimaryKeySelective` + `updateByExampleSelective`

对于以上方法我们进行如下的说明:

* `select` `update` `delete` `insert` 对应了CRUD四种操作
* `byPrimaryKey` `byExample` 对应了 **根据ID** 和 **根据示例(条件)** 执行操作
* `selective` 的含义就是 **选择性的**, 也就是说 只有当我们传入的JavaBean中的属性不为null时才进行更新或者插入...

最后 我们需要说明的其实就是`XxxxExample`的使用方法:

```java
// 查找出工资>2000有奖金的 或者 工资>3000的
EmpExample example = new EmpExample();			// 创建对象
example.setOrderByClause("sal");				// 设置排序
example.setDistinct(true);						// 设置去除重复
EmpExample.Criteria criteria1 = example.createCriteria();		// 创建第一个条件
criteria1.andSalGreaterThan(2000.0f);
criteria1.andCommIsNotNull();
EmpExample.Criteria criteria2 = example.createCriteria();		// 创建第二个条件
criteria2.andSalGreaterThan(3000.0f);
example.or(criteria2);		// 第一个条件 OR 第二个条件
System.out.println(this.empMapper.countByExample(example));
this.empMapper.selectByExample(example).forEach(System.out::println);
```

可以看出 这就是QBC的简化版... Mybatis还是向SpringData发展了...























