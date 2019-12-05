# SpringBoot



## SpringBoot入门

### SpringBoot简介

> 简化SpringBoot应用开发的一个框架 是整个SpringBoot技术栈的一个再整合 所以说SpringBoot是JavaEE开发的一站式解决方案

### 微服务

微服务是一种架构风格 提倡我们在开发一个应用的时候 应该是一组小型服务的集合 可以通过HTTP的方式进行沟通 



### 快速开始

首先我们需要对我们的POM进行配置:

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.2.RELEASE</version>
    <relativePath/>
</parent>

<properties>
    <java.version>1.8</java.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

然后编写一个主程序类:

```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

`@SpringBootApplication`注解用来说明这个类代表的是一个SpringBoot应用 只需要运行这个类的main方法即可运行这个程序

### 将应用打成jar包

如果我们想要将我们的应用打成jar包 只需要添加如下插件:

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
```



### 目录结构

* `/java`: 源代码存放的位置 有了配置类之后果然有些事情变得很微妙呢
* `/resources`: 存放程序相关的资源 主要是配置文件
* `/resources/static`: 静态资源的存放位置
* `/resources/templates`: 模板文件的存放位置
* `/resources/application.properties`: SpringBoot应用的核心配置文件



## SpringBoot原理

### 场景启动器

首先 我们的SpringBoot应用都会继承自一个父项目 这是Maven当中的继承关系 父项目可以帮助我们进行版本控制:

```xml
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-parent</artifactId>
```

如果我们点进去 就会发现 其实`spring-boot-starter-parent`继承自`spring-boot-dependencies`项目 而这个项目中定义了我们JavaEE项目可能需要用到的所有jar包的**版本信息** 以及 **依赖管理** 也就是我们以前建立项目常做的一个事情 在父项目当中进行版本的统一管理:

```xml
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-dependencies</artifactId>
......
<properties>
    <activemq.version>5.15.8</activemq.version>
    <antlr2.version>2.7.7</antlr2.version>
    <appengine-sdk.version>1.9.71</appengine-sdk.version>
    <artemis.version>2.6.3</artemis.version>
    <aspectj.version>1.9.2</aspectj.version>
    <assertj.version>3.11.1</assertj.version>
    ......
</properties>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>jaxen</groupId>
            <artifactId>jaxen</artifactId>
            <version>${jaxen.version}</version>
        </dependency>
        <dependency>
            <groupId>joda-time</groupId>
            <artifactId>joda-time</artifactId>
            <version>${joda-time.version}</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
        </dependency>
        ......
	<dependencies>
<dependencyManagement>
```

然后我们的项目又去依赖了各种`场景启动器(starter)` 这些场景启动器依赖了某个具体场景所需要的所有jar包 这样我们只需要导入某个场景启动器即可 而不用关心具体需要使用什么技术:

```xml
// ::: 我们的项目 :::
<dependencies>
	// 我们是一个WEB项目
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    // 我们需要进行测试
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

至于SpringBoot有什么场景启动器 可以参考[这里](https://docs.spring.io/spring-boot/docs/1.5.19.RELEASE/reference/htmlsingle/#using-boot-starter)



### 内嵌的Tomcat服务器

为什么可以不用部署到tomcat服务器? 因为内置了... 当我们执行`SpringApplication@run()`方法的时候 就相当于去==编译我们的项目然后发布到tomcat服务器并且启动服务器==



### 配置类

---

首先我们要看一下我们启动程序的一句代码 自然是main方法了 java程序的入口:

```java
public static void main(String[] args) {
    SpringApplication.run(SpringbootApplication.class, args);
}
```

`SpringApplication@run(Class, String[])`这个方法要求我们传入一个被`@SpringBootApplication`注解修饰的类 `@SpringBootApplication`注解本身就是一个`@Configuration` 所以说该方法需要传入的实际上是一个`配置类` 传入了配置类 那么接下来就会启动SpringBoot应用内置的tomcat/jetty服务器了 配置类就取代了我们Spring应用中配置文件的作用 那么我们要研究的就是`@SpringBootApplication`注解到底做了什么配置:

```java
@SpringBootConfiguration
@EnableAutoConfiguration
public @interface SpringBootApplication { ... }
```

> 首先 我们可以发现如下的一条线路:
> ```java
> @SpringBootApplication <== @SpringBootConfiguration <== @Configuration
> ```

上面这条线路可以说明我们被`@SpringBootApplication`注解修饰的类就会是一个配置类 一个配置类无法就是配置一些Bean到容器当中 手段无非就是`@Component` `@Bean` `@Import`....



### 自动配置包

---

```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration { ... }
-----------------------------------------
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage { ... }
```

> 然后 我们可以发现如下一条线路:
> ```java
> @SpringBootApplication <== @EnableAutoConfiguration <== @AutoConfigurationPackage
  	  <== @Import(AutoConfigurationPackages.Registrar.class)
> ```

这条路线 如名字说明的一样 功能就是进行**自动配置包** 那么如何做到的呢? 取决于它通过`@Import`注解导入的如下组件: `AutoConfigurationPackages.Registrar` 这个类实现了`ImportBeanDefinitionRegistrar`接口 这个意味着它可以通过注册中心来随心所欲地注册一些Bean:

```java
static class Registrar implements ImportBeanDefinitionRegistrar {
	@Override
	public void registerBeanDefinitions(AnnotationMetadata metadata,
				BeanDefinitionRegistry registry) {
        // 下面这行代码就是导入 被标注的类所在的包中 所有的组件
		register(registry, new PackageImport(metadata).getPackageName());
	}
}
```

我们的应用如果被标注了`@AutoConfigurationPackage`标注了 那么我们的应用**所在的包及其下面的而所有子包**都会被扫描到Spring容器当中 所以说我们入门程序中书写的Controller会被扫描进去 而不用我们去进行手动配置组件扫描了...



### 导入自动配置类

---

```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration { ... }
```

> 最终 我们还有如下一条线路:
> ```java
> @SpringBootApplication <== @EnableAutoConfiguration 
  	  <== @Import(AutoConfigurationImportSelector.class)
> ```

这条线路上 我们可以发现 就是通过`@Import`标签导入了一个组件`AutoConfigurationImportSelector` 从名字上不难看出这个类一定实现了`ImportSelector`接口 这意味着他会通过`selectImports:String[]`方法选择一些组件添加到容器当中:

```java
public class AutoConfigurationImportSelector implements ImportSelector {
	@Override
	public String[] selectImports(AnnotationMetadata annotationMetadata) {
		return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
	}
}
```

上面这个类会导入非常多的**自动配置类** 具体会导入哪些自动配置类 是根本我们使用到了哪些**场景启动器** 这些自动配置类会自动帮我们向容器中注入具体场景中需要的Bean 并且自动配置好这些组件 从而实现自动配置... 那么要到哪里才能找到这些自动配置类呢? 答案是在每个jar包的类路径下的`/META-INF/spring.factories`文件当中(这个文件就是一个键值对映射)寻找键为`EnableAutoConfiguration全类名`的值(值是很多个类的全类名) 然后将这些类导入到容器当中.... 下面我们给出某个`spring.factories`文件的内容:

```properties
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
.....
```

所以说 最终的思路就是:

我们导入了某个**场景启动器** ==> 这个场景启动器的jar包中会有spring.factories文件指定了一些**自动配置类** ==> @EnableAutoConfiguration注解会帮助我们导入 所有场景启动器中的自动配置类 ==> 这些自动配置类会配置好该场景的所有配置



### 自动配置类示例

我们以众多配置类中的一个配置类为例进行说明: `HttpEncodingAutoConfiguration`:

```java
@Configuration
@EnableConfigurationProperties(HttpEncodingProperties.class)
@ConditionalOnWebApplication
@ConditionalOnClass(CharacterEncodingFilter.class)
@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled")
public class HttpEncodingAutoConfiguration {
    @Autowired
	private final HttpEncodingProperties properties;
	@Bean
	@ConditionalOnMissingBean(CharacterEncodingFilter.class)
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
		filter.setEncoding(this.properties.getCharset().name());
		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
		return filter;
	}
}
```

* `@Configuration`: 说明这是一个配置类, 当然了 也是一个组件
* `@EnableConfigurationProperties(HttpEncodingProperties.class)`: 稍后章节说明
* `@ConditionalOnXxxxx`: 这一套系列的注解是说明 当前的组件要在某种必要条件之下才可以注入到容器
* `@Autowired HttpEncodingProperties properties`: 从容器当中获取属性文件
* `@Bean CharacterEncodingFilter`: 用来向容器当中添加字符编码的过滤器 当然要符合某种条件

在上面这些内容当中 我们要说明的就是`@EnableConfigurationProperties(Xxx.class)`:

```java
@Import(EnableConfigurationPropertiesImportSelector.class)
public @interface EnableConfigurationProperties {
	Class<?>[] value() default {};
}
```

说到底 这个注解就是向容器当中注入了一个`EnableConfigurationPropertiesImportSlector`这个`ImportSelecor`会将传入的`Xxx.class`注入到容器当中 而我们传入的正是`HttpEncodingProperties`:

```java
@ConfigurationProperties(prefix = "spring.http.encoding")
public class HttpEncodingProperties {
	private Charset charset = DEFAULT_CHARSET;
    //~ setter, getter
}
```

可以看出`HttpEncodingProperties`这个类并没有注入到容器中 但是被`@ConfigurationProperties`标注了 所以如果这个类一旦被放入到容器当中以后 就会从SpringBoot的核心配置文件当中读取`spring.http.encoding`开头的相关配置 然后注入到属性当中.

最终 `HttpEncodingProperties` 会注入到 `HttpEncodingAutoConfiguration` 当中, 然后供其通过 `@Bean` 进行自动配置相关Bean来使用... 例如==filter.setEncoding(this.properties.getCharset().name());==



### @Conditional系列

* `@ConditionalOnJava`: 判断Java版本是否符合某种需求
* `@ConditionalOnWebApplication`: 要求是Web项目 
* `ConditionalOnNotWebApplication`: 要求不是Web项目
* `@ConditionalOnBean`: 判断容器中是否含有某个Bean
* `@ConditionalOnMissingBean`: 判断容器当中是否不存在某个Bean
* `@ConditionalOnExpression`: 判断是否满足指定的SpEL表达式
* `@ConditionalOnSingleCandidate`: 判断容器中是否有唯一的Bean
* `@ConditionalOnClass`: 判断某个java类是否存在
* `@ConditionalOnMissingClass`: 判断某个java类是否不存在
* `@ConditionalOnResource`: 判断类路径下是否有指定的资源文件
* `@ConditionalOnProperty(prefix="")`: 判断配置文件中是否存在某个前缀



### DEBUG

如何快速查看我们的SpringBoot应用当中导入了哪些自动配置类呢? 可以通过在核心配置文件当中:

`application.properties`

```properties
debug=true
```

然后在我们的SpringBoot应用运行的时候 就可以在控制台上看到 生效的配置类和没有生效的配置类:

```properties
=========================
AUTO-CONFIGURATION REPORT
=========================
Positive matches:
-----------------
......
Negative matches:
-----------------
......
```





## 配置文件

SpringBoot可以使用两种配置文件 分别是`properties`文件 和`yaml`文件 我们需要学习一下yaml的语法...

### YAML语法

* 空格表示缩进
* 必须使用`键 冒号 空格 值`的语法 ==空格不能省略==
* 大小写敏感

接下来我们会说明各种值的写法:

#### 普通类型

```yaml
bool: true
num: 123
str1: abc       # 可以省略引号
str2: '\temp'   # 单引号没有转义字符
str3: "\temp"   # 双引号有转义字符
```

#### 对象/Map类型

```yaml
# 多行写法
user1:
  name: faene
  age: 18
  
# 单行写法
user2: {name: faene, age: 18}
```

#### 数组/列表类型

```yaml
# 多行写法
pets1:
  - cat
  - dog
  - pig

# 单行写法
pets2: [cat, dog, pig]
```



### @ConfigurationProperties

我们向容器中注入组件的时候 往往还会将其依赖的组件进行注入 也就是DI(依赖注入) 我们进行依赖注入的方式有很多种 例如`@Value`注解 `@Autowire`注解 等等 那么我们如何才能==将配置文件中配置好的值注入到组件当中呢== 我们可以通过`@ConfigurationProperties`注解来实现这个操作:

1. 将我们的组件使用`@ConfigurationProperties`注解进行修饰 这意味着这个组件的所有属性都会从配置文件中获取 `prefix=""`属性用来指定 用哪个节点作为根节点来进行属性注入:

```java
@Component										// 说明这是一个组件
@ConfigurationProperties(prefix = "person")		// 说明这个组件通过配置文件进行属性注入
public class Person {
    private String name;
    private Integer age;
    private List<Pet> pets;
    ......
}
```

2. 在SpringBoot的唯一指定配置文件: `application.properties`或者`application.yaml`当中进行配置:

```yaml
person:
  name: faene
  age: 24
  pets:
    -
      name: abc
      age: 18
    - {name: def, age: 20}
```

3. 进行测试 检测容器中的组件是否为配置文件中配置好的属性:

```java
@RunWith(SpringRunner.class)			// SpringBoot整合Junit需要使用的类
@SpringBootTest							// 不需要指定配置文件位置了 因为结构是固定的
public class YamlTestApplicationTests {
    @Autowired
    private Person person;
    @Test
    public void contextLoads() {
        System.err.println(this.person);
    }
}
```

> **松散绑定**
>
> 需要说明的是 我们使用+`@ConfigurationProperties`进行属性注入的时候 我们配置文件中属性名称的书写可以更加"泛化":
>
> ```yaml
> person:
> first-name: billow
> last-name: pang
> ```
>
> 在我们的java中 类的属性可不会出现"first-name"这种形式的写法 但是这不影响我们在配置文件中如上书写.
>
> **SpEL**
>
> 不过`@ConfigurationProperties`是不支持SpEL的... 相比于`@Value`...



**没有提示问题**

如果我们发现我们属性对象属性的时候没有响应的提示信息 那么我们就需要导入如下的依赖:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```



### @PorpertySource

这个注解我们已经很熟悉了 就是扫描一个`.properties`文件 然后在我们的配置文件或者`@Value`注解中 就可以使用`${key}`的语法来引用了 我们要强调的一点是:

==**SpringBoot已经默认将核心配置文件application.propertis引用了 也就是说可以直接通过${key}来获取值**==

但是我们不会把所有的配置都放入到核心配置文件当中 所以说 我们需要引用其他的配置文件 `@PropertySource`注解可以添加在任意的组件之上 都是可以生效的...

```java
@Component
@ConfigurationProperties(prefix = "person")
@PropertySource("classpath:props/person.properties")
public class Person {
    private String name;
    @Value("${person.age}")
    private String age;
...
```



### **${key}**的妙用

* `${random.int}` `${random.int(10)}` `${random.int[1024, 65535]}` `${radnom.uuid}`

* ```properties
  jdbc.username=root
  jdbc.userdesc=username is ${jdbc.username}, password is ${jdbc.password:root}
  ```

  * `${jdbc.username}`: 引用已经配置好的变量的值
  * `${jdbc.password:root}`: 如果引用的值不存在 使用默认值root

c

### @ImportResource

这个注解的作用是导入Spring的XML配置文件 在SpringBoot当中==不建议再使用Spring配置文件 而应该使用配置类== 我们在`@SpringBootApplication`标注的类所在的包中创建`@Configuration`类即可... 由于`@AutoConfigurationPackge`的作用 我们的配置类会被扫描到容器当中.... 从而发挥作用, 如果一定要使用配置文件 可以:

```java
@SpringBootApplication
@ImportResource(locations = {"classpath:config/beans.xml"})
public class Springboot02Application {
    public static void main(String[] args) {
        SpringApplication.run(Springboot02Application.class, args);
    }
}
```



### 配置文件加载顺序

SpringBoot会从下面四个位置依次加载名为`application.properties/yaml`的配置文件 后加载的配置文件优先级更高 会覆盖掉优先级低的相关配置...

==`project:/config/` ==> `project:/` ==> `classpath:/config/` ==> `classpath:/`==

需要注意的是`project:/`路径下指的就是 我们的`项目路径`下 也就是说==application.yaml与src同级== 而类路径下就没什么好说的了... `/src/java`和`/src/resources`都是类路径

1. 
2. b
3. 







## 日志

日志可以分为日志门面和日志实现:

* `日志门面`: JCL(Jakarta Commons Logging), **SLF4j**, <del>JBoss-Logging</del>
* `日志实现`: Log4j, Log4j2, JUL(java.util.logging), **Logback**

我们选择的日志门面都应该是SLF4j 因为他各方面都是更加优秀的, 对于日志实现来说 我们应该选择Logback, SpringBoot与我们的选择是一样的



### **整合不同的日志实现**

![click to enlarge](https://www.slf4j.org/images/concrete-bindings.png)

通过官方提供的这张图 我们已经非常清晰了 在日志`实现层` 和日志`门面层`之间我们引入了日志`适配层`:

* 整合`Logback`: 无需适配层 直接使用即可
* 整合`Log4j`: 需要提供适配层 `slf4j-log412`
* 整合`JUL`: 需要提供适配层`slf4j-jdk14`

### **统一已有的日志框架**

![img](https://www.slf4j.org/images/legacy.png)

对于我们的应用程序 可能已经使用了某种框架 这些框架使用的日志框架不一定是我们程序要使用的框架 这个时候我们就需要将我们的日志框架统一成SLF4J 根据官方给出的配置图 我们需要以假换真, 将一些其他日志框架的API替换为新的API:

* 替换`JCL`: 需要我们导入`jcl-over-slf4j`
* 替换`log4j`: 需要我们导入`log4j-over-slf4j`
* 替换`JUL`: 需要我们导入`jul-to-slf4j`

如果我们使用的框架用到了上面的几种日志框架 那么就需要引入上面对应的包来统一到SLF4J:

* `Spring`
* `Hibernate`
* `Mybatis`
* `Struts2`

### SLF4J API

```java
public class Slf4j_Test {
    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(Slf4j_Test.class);
        logger.trace("trace");
        logger.debug("debug");
        logger.info("info");
        logger.error("error");
        logger.warn("warn");
    }
}
```

可以看出SpringBoot使用的日志分为了五个级别: `trace` `debug` `info` `warn` `error` `off`



### SpringBoot日志配置

![1553581138704](assets/1553581138704.png)

在SpringBoot当中 默认的日志级别是`INFO` 我们可以通过配置文件进行调整:

* `logging.level.root=OFF`: 设置默认的日志级别
* `logging.level.me.faene.spring=TRACE`: 设置指定包的日志级别
* `logging.file=springboot.log`: 指定日志文件 和`logging.path`二选一
  * `logging.path=/springboot/log`: 在指定的路径生成日志文件
* `logging.pattern.console`: 指定控制台输出日志的格式
* `Logging.pattern.file`: 指定文件输出日志的格式

```properties
# 示例配置
logging.level.root=info				# 主日志级别
logging.level.me.faene=info			# 指定包的日志级别
logging.file=springboot.log			# 配置文件指定, 和下面二选一
#logging.path=/springboot/log		# 配置文件位置指定, 和上面二选一
logging.pattern.console=[%-5level] - %msg%n		# 控制台输出格式
logging.pattern.file=%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n
```



### SpringBoot日志原理

由于SpringBoot底层使用了`SLF4J` + `Logback`作为日志框架, 所以说日志的配置文件应该为Logback的日志配置文件, 但是为什么我们却可以配置在`application.properties`当中呢? 

==spring-boot.jar/org/springframework/boot/logging/logback==目录当中存放了若干个Logback的日志配置文件:

* `base.xml`: 基本的日志配置文件
* `defaults.xml`: 默认的日志配置文件
* `console-appender.xml`: 控制台输出的日志配置文件
* `file-appender.xml`: 文件输出的日志配置文件

在`base.xml`当中 我们可以发现如下的配置:

```xml
<included>
	<include resource="org/springframework/boot/logging/logback/defaults.xml" />
	<property name="LOG_FILE" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}}/spring.log}"/>
	<include resource="org/springframework/boot/logging/logback/console-appender.xml" />
	<include resource="org/springframework/boot/logging/logback/file-appender.xml" />
	<root level="INFO">
		<appender-ref ref="CONSOLE" />
		<appender-ref ref="FILE" />
	</root>
</included>
```

* `<include resource="...">`: 引入了三个其他的配置文件
* `<root level="INFO">`: 设置了文件和控制台的默认日志级别为`INFO`

---

在这四个配置文件当中 我们不难发现 有很多的`${...}`语法来获取某些变量值 那么他们获取的是哪里的变量值呢? 我们需要了解下面的两个类:

`LoggingApplicationListener` 和 `LoggingSystemProperties` 的代码如下:

```java
class LoggingSystemProperties {
	String CONSOLE_LOG_PATTERN = LoggingApplicationListener.CONSOLE_LOG_PATTERN;
	String FILE_LOG_PATTERN = LoggingApplicationListener.FILE_LOG_PATTERN;
	String LOG_LEVEL_PATTERN = LoggingApplicationListener.LOG_LEVEL_PATTERN;
    public void apply(LogFile logFile) {
        RelaxedPropertyResolver propertyResolver = RelaxedPropertyResolver
            .ignoringUnresolvableNestedPlaceholders(this.environment, "logging.");
        setSystemProperty(propertyResolver, CONSOLE_LOG_PATTERN, "pattern.console");
        setSystemProperty(propertyResolver, FILE_LOG_PATTERN, "pattern.file");
        setSystemProperty(propertyResolver, LOG_LEVEL_PATTERN, "pattern.level");
}
```

从上面这段代码当中不难看出 这两个类本质上就是从`application.properties`获取例如`logging.pattern.file`变量的值, 然后存储到自身的成员变量当中`FILE_LOG_PATTERN`, 最终又会被`file-appender.xml`中的`${FILE_LOG_PATTERN}`引用:

```xml
<included>
	<appender name="FILE"
		class="ch.qos.logback.core.rolling.RollingFileAppender">
		<encoder>
			<pattern>${FILE_LOG_PATTERN}</pattern>
		</encoder>
		...
}
```



### SpringBoot原生日志

如果我们想要使用原生的Logback的日志配置文件 需要怎么做呢? 官方给出了如下的列表:

* **Logback**: `logback.xml` `logback-spring.xml`
* **Log4j2**: `log4j2.xml` `log4j2-spring.xml`
* **JUL**: `logging.properties`

一旦我们在类路径下放入了如上的配置文件 那么SpringBoot就不会使用默认的配置文件了.... 官方建议我们放带有`-spring`后缀的日志配置文件 这样可以拥有Spring相关的特性



## Web开发

### 静态资源文件夹

SpringBoot提供了四个静态资源文件夹 任何jar包 包括主应用中的如下目录均为静态资源文件夹:

* `classpath:/public/**`
* `classpath:/static/**`
* `classpath:/resources/**`
* `classpath:/META-INF/resources/**`



### Webjars

[webjars](https://www.webjars.org/)就是一个提供各种常见的静态资源集合的jar包 他们拥有统一的规范 以Jquery的webjar为例:

```xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.0.0</version>
</dependency>
```

它jar包当中的目录结构如下:

* `jquery-3.0.0.jar`/`META-INF`/`resources`/`webjars`/`jquery`/`3.0.0`/`jquery.js`

只要我们导入了上面的webjar, 就可以通过浏览器访问到jar包中的资源了... 映射规则如下:

`http://localhost:8080/contextPath/webjars/**` ===> `webjar.jar/META-INF/resources/webjars/**`

![1553596263903](assets/1553596263903.png)



### 首页 & 图标

SpringBoot应用的首页就是我们静态资源文件夹中的`index.html`

SpringBoot应用的图标就是我们静态资源文件夹中的`favicon.ico`



### 静态资源配置原理

我们刚才所描述的静态资源文件夹也好, Webjars也好, 以及首页/图标的配置其实都是默认配置在了`WebMvcAutoConfiguration`配置类当中:

```java
public class WebMvcAutoConfiguration {
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        // 添加Webjars的静态资源映射
        registry
            .addResourceHandler("/webjars/**")
            .addResourceLocations("classpath:/META-INF/resources/webjars/");
        // 添加静态资源文件夹
        String staticPathPattern = this.mvcProperties.getStaticPathPattern();
        registry
            .addResourceHandler(staticPathPattern)
            .addResourceLocations(getResourceLocations(
                this.resourceProperties.getStaticLocations()));
        /*  上面三行代码的翻译版本:
        	.addResourceHandler("/**")
            .addResourceLocations(new String[] {
                "classpath:/META-INF/resouces/",
                "classpath:/resources/",
                "classpath:/static/",
                "classpath:/public/"
            });
        */
    }
}
```

可以看出, `WebMvcAutoConfiguration`向容器中添加了若干个`ResourceHandler`, 而这个`ResourceHandler`对应了SpringMVC配置文件当中的`<mvc:resources mapping="" location="">`:

* `addResourceHandler(String)` = `mapping=""`
  * 定义Web的访问路径, 例如`/**`, 也就是根路径下的所有访问.
* `addResourceLoacaitons(String...)` = `location=""`
  * 定义指定的访问路径要到哪里去寻找资源, 例如`/abc`, 也就是到这个文件夹里去找.

而这个配置类配置了两个内容, 从而使得我们的静态资源路径问题得以解决:

* `/webjars/**` ==> `classpath:/META-INF/resources/webjars/`
* `/**` ==> `classpath:/META-INF/resouces/` 等... 共4个静态资源文件夹
  * application.yml当中的`spring.mvc.staticPathPattern:String`可以设置左侧的值
  * applicaiton.yml当中的`spring.resouces.staticLocations:String[]`
* 



### 使用Thymeleaf

如果在SpringBoot当中想要使用Thymeleaf模板引擎 仅仅需要我们导入相关的场景启动器即可:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

由于我们学习的是SpringBoot1.5版本 这个时候引入的Thymeleaf包版本为2.1.6 这个版本太低了 我们需要手动将其提高版本... 只需要配置两个properties即可:

```xml
<thymeleaf.version>3.0.9.REALEASE</thymeleaf.version>
<thymeleaf-layout-dialect.version>2.2.2</thymeleaf-layout-dialect.version>
```

3版本的thymeleaf对应了2版本的layout, 2版本的thymeleaf对应了1版本的layout



**请求转发的路径问题**

Thymeleaf使用`HTML`作为模板 而非JSP 而在SpringBoot当中也有SpringMVC视图解析器中的前缀和后缀的相关概念, ==SpringBoot当中的模板都应该放在`classpath:templates`当中== 而原因嘛 就是因为:

`prefix` + `path` + `suffix` === `classpath:/templates/` + `path` + `.html`

所以说 我们应该如下操作:

```java
@Controller
public class TestController {
    @RequestMapping("/test")
    public String test(Model model) {
        model.addAttribute("name", new String("FAENE"));
        model.addAttribute("age", new Integer(20));
        model.addAttribute("sex", new Boolean(true));
        model.addAttribute("hobby", Arrays.asList("ACGN".split("")));
        return "test";
    }
}
```

上面的处理器方法会将页面转发到`classpath:templates/ test .html`



接下来 我们只讲解最基本的 如何从request域当中获取值显示出来, 详细的用法请参考其他手册 但是无论如何我们使用Thymeleaf的第一步都应该是:

**导入Thymeleaf命名空间 这样就有提示了**

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8"/>
    <title>Title</title>
</head>
...
```

**最基本的获取模型数据方式**

```html
<div>
    <span th:text="${name}"></span>
    <span th:text="${age}"></span>
    <span th:text="${sex}"></span>
    <span th:text="${hobby}"></span>
</div>
```



### Thymeleaf配置原理

`ThymeleafAutoConfiguration` & `ThymeleafProperties`

这个配置类会向容器中注入`ThymeleafViewResolver`, 这个视图解析器和`InternalResourceViewResolver`一样, 可以配置访问的前缀以及后缀, 不过不是从配置文件中获取值, 而是从`ThymeleafProperties`当中去获取到值:

* `spring.thymeleaf.prefix:String` = `classpath:/templates/`
* `spring.thymeleaf.suffix:String` = `.html`

所以说, 最终的模板路径为: ==*classpath:templates/*`???`.html==



## 数据访问



### SQL

SpringBoot无论去整合任何一种关系型数据访问的技术, ==都应该导入数据库驱动包, 而连接池则是可有可无的==.

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.27</version>
    <scope>runtime</scope>
</dependency>
```



### 场景启动器

数据访问相关的场景启动器如下:

* `spring-boot-starter-jdbc`

* `spring-boot-starter-data-jpa`
* `spring-boot-starter-data-mongodb`
* `spring-boot-starter-data-redis`



### SpringBoot & JDBC











### SpringBoot & Mybatis











### SpringBoot & SpringData JPA







## 消息队列

### 概述

* 大多数应用当中, 可以通过消息服务中间件来提升系统**异步通信**, **扩展解耦**能力

  * 异步处理: 注册消息=>发送邮件&发送短信
  * 应用解耦: 订单系统调用库存系统, 需要发布和订阅
  * 流量削峰: 将用户请求写入消息队列, 然后异步执行

* 消息服务中两个重要的概念:

  * **消息代理(message broker)**: 消息发送者发送消息以后, 将有消息代理接管

  * **目的地(destination)**: 消息队列主要有两种形式的目的地

    * **队列(queue)**: 点对点(P2P)通信

      消息发送者发送消息 => 消息代理将其放入到队列中 => 消息接受者从队列中获取消息(POP)

      消息只有唯一的发送者和接受者, 但是并不是说只有一个接受者

    * **主题(topic)**: 发布(publish)和订阅(subscribe)消息通信

      发送者发送消息到主题, 多个接受者(订阅者)监听订阅这个主题, 那么就会在消息到达的时候同时收到消息

* 常见的消息队列的规范有:
  * **JMS(java message service)**: 基于JVM的消息代理的规范, *ActiveMQ*和*HornetMQ*是JML的实现
  * **AMQP(advanced message queuing protocol)**: 高级消息队列协议, 也是一个消息队列的规范,  兼容JMS, *RabbitMQ*就是实现之一

| 规范名称 |      实现技术      |   消息模型   | 跨语言 | 跨平台 |     定义     |
| :------: | :----------------: | :----------: | :----: | :----: | :----------: |
|   JMS    | ActiveMQ, HornetMQ | P2P, Pub/Sub |   否   |   否   |   JavaAPI    |
|   AMQP   |      RabbitMQ      |    有五种    |   是   |   是   | 网络线级协议 |

* Spring支持情况概述
  * `spring-jms`提供了对JMS规范的支持
  * `spring-rabbit`提供了对AMQP规范的支持
  * 需要使用`ConnectionFactory`的实现来连接消息代理
  * `JmsTemplate` & `RabbitTemplate`来发送消息
  * `@JmsListener` & `RabbitLisener` 注解在方法上监听消息代理发布的消息
  * `@EnableJms` & `EnableRabbit` 开启支持
* SpringBoot的情况
  * 通过`JmsAutoConfiguration` & `RabbitAutoConfiguration` 来进行自动配置





### RabbitMQ

RabbitMQ是一个利用**Erlang**语言开发的**AMQP**开源实现, 其中有一些核心概念需要我们来掌握:

* `消息(message)`: 分为消息头(一系列的可选属性)和消息体
* `发布者(Publisher)`: 消息的生产者 也是一个向交换器发布消息的客户端应用程序
* `交换器(Exchange)`: 用来接收发布者发送的消息并将这些消息路由给服务器中的队列
* `队列(Queue)`: 消息的容器, 用来保存消息直到消费者获取到
* `绑定(Binding)`: 用于==消息队列和交换器之间的关联关系==, 一个绑定就是基于路由键将交换器和队列连接起来的规则, 所以可以将交换器理解为==由绑定构成的路由表==, 而交换器的和队列的绑定可以是多对多的.
* `连接(Connection)`: 如果要操作消息队列我们需要和队列之间建立起连接
* `信道(Channel)`: 信道就是建立在TCP连接内的虚拟连接, 因为建立和销毁TCP都是代价昂贵的, 所以要使用信道
* `消费者(Consumer)`: 表示从消息队列中取得消息的客户端应用程序
* `虚拟主机(VirtualHost)`: 虚拟主机就是mini版的RabbitMQ服务器, 拥有各自的队列, 交换器, 绑定以及权限机制, 我们在连接RabbitMQ的时候, 必须要指定一个虚拟主机, 默认的虚拟主机是`/`

* `消息代理(Broker)`: 消息代理指的就是管理队列的服务器

![1554641356606](assets/1554641356606.png)



### RabbitMQ运行机制

* *Publisher*将消息发送到*Broker*服务器中某个*VirtualHost*的某个*Exchange*, *Exchange*和*Queue*之间存在着多对多的*Binding*关系, *Exchange*就根据*Message*的路由键来决定将其发送到哪个*Queue*当中, 然后*Consumer*就可以利用*Channel*和*Broker*之间建立*Connection*, 获取其中的*Message*.



### Exchange

* Exchange分发消息的时候根据类型的不同分发的策略是有所区别的, 总共有四种类型:
  * **direct**: 消息中的路由键(routing key)如果和绑定中的绑定键(binding key)一致, 就会发送到指定队列.
  * **fanout**: 其实就是广播模式, 当消息达到交换器的时候, 直接发送到所有的队列当中, 就是**广播**, 最快
  * **topic**: 根据消息中的路由键(routing key)符合哪种模式, 从而转发到对应的队列, 支持通配符:
    * `#`: 匹配一个或者多个单词
    * `*`: 匹配一个单词

