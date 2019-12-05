# Spring XML



## API

```java
// 单例(默认)不是延迟加载 多例延迟加载
ApplicationContext applicationContext = new ClassPathXmlApplicationContext(
    	"classpath:applicationContext.xml"
);
// 延迟加载 无论是单例还是多例
BeanFactory beanFactory = new XmlBeanFactory(new ClassPathResource(
        "applicationContext.xml"
));
```





## IoC

### Bean的3种实例化方式

* 默认构造: 

```xml
<bean id="person" class="me.faene.spring.bean.Person"/>
```

* 静态工厂: 

```java
public class PersonStaticFactory {
    public static Person getObject() {						// 需要提供静态方法
        return new Person();
    }
}
```

```xml
<bean id="person"
      class="me.faene.spring.bean.PersonStaticFactory"		// 指定静态工厂全类名
      factory-method="getObject"							// 指定静态方法名称
/>
```

* 实例工厂

```java
public class PersonInstanceFactory {
    public Person getObject() {								// 需要提供实例方法
        return new Person();
    }
}
```

```xml
// 首先 需要将实例工厂配置成Bean
<bean id="personInstaceFactory" class="me.faene.spring.bean.PersonInstanceFactory"/>
// 然后 通过如下方式获取到实例工厂创建的Bean
<bean id="person"
      factory-bean="personInstaceFactory"					// 实例工厂Bean的ref
      factory-method="getObject"							// 指定实例方法名称
/>
```



### Bean的种类

* 普通Bean

```java
public class Person {}
```

```xml
<bean id="person" class="me.faene.spring.bean.Person"/>
```

* FactoryBean

```java
// 必须实现 FactoryBean<T> 接口来说明这是一个FactoryBean
public class PersonFactoryBean implements FactoryBean<Person> {
    @Override
    public Person getObject() throws Exception {			// 工厂用于创建对象的方法
        return new Person();
    }
    @Override
    public Class<?> getObjectType() {
        return Person.class;
    }
    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

```xml
// 虽然配置的全类名是FactoryBean 但是实际Spring容器创建的对象是工厂生产的对象
<bean id="person" class="me.faene.spring.bean.PersonFactoryBean"/>
```



`小思考`

> **FactoryBean<T&gt;为什么不看作是一种Bean的实例化方式? 为什么要看做一种类型的Bean?**
>
> 答: 因为你无论你用什么方式注入一个FactoryBean 最终向容器中存储的都是工厂生产的对象的类型的Bean
>
> ```java
> public class PersonFactoryBeanStaticFactory {
>     public static PersonFactoryBean getOjbect() {
>         return new PersonFactoryBean();
>     }
> }
> ```
>
> ```xml
> <bean id="person"
>       class="me.faene.spring.bean.PersonFactoryBeanStaticFactory"
>       factory-method="getOjbect"
> />
> ```
>
> 上面这种方式 我们就可以说: 我们使用的是`静态工厂`的方式注册一个Bean 我们注册的Bean的类型是`FactoryBean` 所以说最终容器中注入的还是Person而非PersonFactoryBean
>
> 
>
> **FactoryBean和BeanFactory的区别是什么呢?**
>
> 答: BeanFactory是已经过时的容器对象 可以通过这个对象获取容器中注册的Bean, 而FactoryBean是Bean的一种类型 如果注册实现了FactoryBean接口的Bean 实际注册的其实是getObject()方法产生的对象



### Bean的作用域

作用域就是说创建Bean是多例还是单例的...

```xml
<bean id="person" class="me.faene.spring.bean.Person" 
	  scope="singleton | prototype"
/>
```

如果是单例的 ApplicationContext会`提前加载` 多例的话则是`延迟加载`, 对于BeanFactory来说全是延迟加载



### Bean的生命周期

1. ==new Bean()==

首先肯定要创建指定的Bean

2. ==BeanPostProcessor # postProcessBeforeInitialization()== 

在Spring容器中有很多默认的 以及自己配置好的`Bean后置处理器` 他们都实现了`BeanPostProcessor`接口 在创建完Bean之后就要去一次执行这些`Bean后置处理器`的`postProcessBeforeInitialization()`方法 也就是`后置处理 在 初始化之前`方法

3. ==Bean # initMethod()==

接下来需要执行Bean的初始化方法了 也就是我们在配置Bean的过程中 `init-method`属性指定的方法

4. ==BeanPostProcessor # postProcessAfterInitialization()==

顾名思义 要执行`Bean后置处理器`的`后置处理在初始化之后`方法...

5. ==Bean # destroyMethod()==

只有当容器关闭的时候才会执行这个方法 也就是Bean的销毁方法 `destroy-method`属性指定的方法

****

`Bean后置处理器(BeanPostProcessor)`的作用是什么呢?

首先它可以在Bean执行初始化方法(init-method)**之前**和**之后**去对这个Bean执行一些操作 例如: **返回一个代理对象** 这也是AOP的底层实现原理 下面我们就来写一个自己的Bean后置处理器

```java
public class MyBeanPostProcesser implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object o, String s) throws BeansException 
    {
        System.out.println("MyBeanPostProcesser.postProcessBeforeInitialization");
        return o;		// 这里没有返回代理对象 而是直接返回原有的对象
    }
    @Override
    public Object postProcessAfterInitialization(Object o, String s) throws BeansException 
    {
        System.out.println("MyBeanPostProcesser.postProcessAfterInitialization");
        return Proxy.xxx...;		// 完全可以使用jdk动态代理返回一个代理对象
    }
}
```

当然我们的Bean后置处理器一定要配置到Spring容器当中才会生效

```xml
<bean class="me.faene.spring.bean.MyBeanPostProcesser"/>
```

---

```java
public class Person {
    public Person() {
        System.out.println("Person.Person");				// 构造器
    }
    public void initMethod() {								// 初始化方法
        System.out.println("Person.initMethod");
    }
    public void destroyMethod() {							// 销毁方法
        System.out.println("Person.destroyMethod");
    }
}
```

```xml
<bean id="person" class="me.faene.spring.bean.Person"
      scope="singleton"										// 保证容器创建 Bean就提前加载
      init-method="initMethod"								// 指定哪个是初始化方法
      destroy-method="destroyMethod"						// 指定哪个是销毁方法
/>
```

---

最终的测试:

```java
public class SpringTest {
    public static void main(String[] args) throws Exception {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext(
                "classpath:applicationContext.xml"
        );
        // 接口中并没有直接定义close()方法 不过实现类中是有的
        applicationContext.getClass().getMethod("close").invoke(applicationContext);
    }
}
```

最终的结果:

```xml
Person.Person
MyBeanPostProcesser.postProcessBeforeInitialization
Person.initMethod
MyBeanPostProcesser.postProcessAfterInitialization
Person.destroyMethod
```



## DI

### 注入方式

* setter注入

```java
public class Person {
    private String name;
    private Integer age;
    public void setName(String name) {		// 需要提供setter 这样才能通过反射进行设置属性
        this.name = name;
    }
    public void setAge(Integer age) {
        this.age = age;
    }
}
```

```xml
<bean class="me.faene.spring.bean.Person" id="person">
    <property name="name" value="小烯"/>			// 可以直接使用value属性
    <property name="age">
        <value>20</value>						// 也可以使用这种写法
    </property>
</bean>
```

* 构造器注入

```java
public class Person {
    private String name;
    private Integer age;
    public Person(String name, Integer age) {			// 需要提供构造器
        this.name = name;
        this.age = age;
    }
}
```

```xml
<bean class="me.faene.spring.bean.Person" id="person">
    <constructor-arg index="0" value="小烯"/>
    <constructor-arg index="1">
        <value>20</value>
    </constructor-arg>
</bean>
```





### 不同类型的注入

* 单个:普通类型

```xml
<bean class="me.faene.spring.bean.Person" id="person">
    // 简写: <property name="name" value="abc"/>
    <property name="name">
        <value>abc</value>
    </property>
</bean>
```

* 单个:引用类型

```xml
<bean class="me.faene.spring.bean.Person" id="other"/>

//这种方式当然可以 通过ref属性
<bean class="me.faene.spring.bean.Person" id="person1">
    <property name="other" ref="other"/>
</bean>

// 这种方法也可以 通过<ref bean="">标签
<bean class="me.faene.spring.bean.Person" id="person2">
    <property name="other">
        <ref bean="other"/>
    </property>
</bean>

// 这种方式是通过内部Bean的形式 达到匿名的效果
<bean class="me.faene.spring.bean.Person" id="person3">
    <property name="other">
        <bean class="me.faene.spring.bean.Person"/>
    </property>
</bean>
```

* 集合类型的注入

```xml
<bean class="me.faene.spring.bean.Person" id="person">
-- array ---------------
    <property name="array">
        <array>
            <!-- 根据元素类型来写若干个<value>或者<ref bean="">或者<bean class=""> -->
        </array>
    </property>
-- list ---------------
    <property name="list">
        <list>
            <!-- 根据元素类型来写若干个<value>或者<ref bean="">或者<bean class=""> -->
        </list>
    </property>
-- set ---------------
    <property name="set">
        <set>
            <!-- 根据元素类型来写若干个<value>或者<ref bean="">或者<bean class=""> -->
        </set>
    </property>
-- map ---------------
    <property name="map">
        <map>
            <entry key="">
                <!-- 根据值类型来写若干个<value>或者<ref bean="">或者<beanh class=""> -->
            </entry>
        </map>
    </property>
-- properties ---------------
    <property name="prop">
        <props>
            <!-- prop的键和值都固定是字符串 所以采用如下写法 -->
            <prop key="str">str</prop>
        </props>
    </property>
</bean>
```



### p名称空间注入

p名称空间注入是一种新的注入方式 特点如下:

* 配置文件引入p名称空间 ==xmlns:p="http://www.springframework.org/schema/p"==

* 提供setter方法(只提供构造器是不可以的)
* 仅支持普通类型和引用类型

然后就可以通过如下的语法来进行注入了:

```xml
<bean class="me.faene.spring.bean.Person" id="person"
      p:name="小烯"					// 普通类型注入
      p:person-ref="p1"				 // 引用类型注入
/>
```



### SpEL注入

SpEL是一种新的注入方式 特点如下:

* 可以看出我们普通的方式对于不同类型的注入语法参差不齐 SpEL可以使我们==对于任何类型的属性注入 都采用`<property name="name" value="SpEL">`的方式书写==

* 既然语法是<property&gt;那么肯定要提供setter了



SpEL的语法:

* 格式: `#{SpEL}`

* 简单类型: `#{10}` `#{3.14}` `#{2e10}` `#{'string'}`
* 引用类型: `#{beanID}`
* 引用Bean的属性: `#{beanID.propName}`
* 引用Bean的方法的返回值: `#{beanID.methodName()}`
* 引用Bean的方法的返回值 不过防止空指针异常: `#{beanID.?methodName()}`
* 引用静态常量的值或方法的返回值: `#{T(全类名).静态常量/静态方法()}`
* 运算符支持, 正则支持, 集合支持... 请自行查询文档



## AOP

AOP就是`面向切面编程 ` 通过预编译的方式或者动态代理的方式 来实现程序功能统一维护的一种技术. 通过横向抽取 取代了 传统的纵向继承, AOP的经典用法有事务管理, 性能监控, 安全检查, 缓存等等
### AOP术语

* `目标类(target)`: 就是需要代理的类
* `连接点(joinpoint)`: 目标类中的可能会被拦截的点(方法)
* `切入点(pointcut)`: 表示一组连接点 通过表达式来集中起来 切入点是连接点的子集
* `切面类`: 就是用于存储各种通知的类 人为定义的
* `通知(advice)`: 加强的代码 表示对于切入点具体要进行的操作
* `拦截器(Interceptor)`: 拥有将通知织入切入点的能力的对象 认为定义的
* `织入(weaving)`: 将通知和切入点结合的过程叫做织入
* `切面(Aspect)`: 切入点和通知的结合就是切面 也就是织入的结果
* `代理类(Proxy)`: 最终的结果类

---



AOP动态代理有两种方式: 

* `JDK动态代理`: 需要提供接口和实现类
* `cglib动态代理`: 通过字节码增强 需要提供实现类即可

### JDK动态代理

```java
public class JdkProxyTest {
    @Test
    public void test() {
        // 目标类 其中有很多连接点
        Target target = new TargetImpl();
        // 切面类 其中有很多通知
        Aspect aspect = new Aspect();
        // 代理类
        Target proxy = (Target) Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args)
                        	throws Throwable {
                        /*
                          proxy: 代理的对象
                          method: 代理对象当前执行的方法
                          args: 方法的参数列表
                         */
                        // 选中若干个方法 相当于从连接点当中选取好了切入点
                        if(method.getName().matches("doit[12]")) {
                            // 将通知织入到切入点当中形成切面
                            aspect.advice1();
                            method.invoke(target, args);
                            aspect.advice2();
                        }
                        return proxy;
                    }
                }
        );
        proxy.doit1();
        proxy.doit2();
        proxy.doit3();
        /* output:
            Aspect.advice1
            TargetImpl.doit1
            Aspect.advice2
            Aspect.advice1
            TargetImpl.doit2
            Aspect.advice2
         */   
    }
}
```

### cglib动态代理

可以单独找cglib的依赖 也可以直接引入spring核心的依赖 它们又会去依赖cglib...

```java
public class CglibTest {
    @Test
    public void test() {
        // 目标类 其中有很多连接点
        Target target = new Target();
        // 切面类 其中有很多通知
        Aspect aspect = new Aspect();
        // 代理类是通过Enhancer这个对象产生的
        Enhancer enhancer = new Enhancer();
        // 设置目标类
        enhancer.setSuperclass(target.getClass());
        // 设置回调函数
        enhancer.setCallback(new MethodInterceptor() {
            @Override
            public Object intercept(Object o, Method method, Object[] args, 
                            MethodProxy methodProxy) throws Throwable {
                Object result = null;
                // 选中若干个方法 相当于从连接点当中选取好了切入点
                if(method.getName().matches("doit[12]")) {
                    // 将通知织入到切入点当中形成切面
                    aspect.advice1();
                    result = method.invoke(target, args);
                    aspect.advice2();
                }
                return result;
            }
        });
        // 获取到代理类对象
        Target proxy = (Target) enhancer.create();

        proxy.doit1();
        proxy.doit2();
        proxy.doit3();
        /*
            Aspect.advice1
            TargetImpl.doit1
            Aspect.advice2
            Aspect.advice1
            TargetImpl.doit2
            Aspect.advice2
         */
    }
}
```



### Spring工厂Bean动态代理

首先需要我们引入`spring-aop`这个依赖 同时它会依赖aop联盟的提供的规范, AOP联盟采用了一种`方法拦截器`的机制来产生代理对象 我们如果想定义一个方法拦截器需要实现`MethodInterceptor`接口:

```java
public class MyMethodInterceptor implements MethodInterceptor {
    @Override
    public Object invoke(MethodInvocation methodInvocation) throws Throwable {
        // 从连接点当中选取切入点
        if(methodInvocation.getMethod().getName().matches("doit[12]")) {
            Aspect aspect = new Aspect();		// 创建切面类
            aspect.advice1();					// 执行通知
            Object proxy = methodInvocation.proceed();		// 执行目标方法
            aspect.advice2();
            return proxy;
        }
        return methodInvocation.proceed();
    }
}
```

然后就可以进行AOP的相关配置了:

```xml
<!-- 配置目标类 -->
<bean id="target" class="me.faene.spring.bean.Target"/>

<!-- 配置方法拦截器 -->
<bean id="interceptor" class="me.faene.spring.bean.MyMethodInterceptor"/>

<!-- 通过Spring提供的ProxyFactoryBean来产生代理 -->
<bean id="proxy" class="org.springframework.aop.framework.ProxyFactoryBean">
    <!-- 有接口还要添加这个属性 会使用jdk动态代理 -->
    <!--<property name="interfaces" value="..."/>-->
    <!-- 设置目标类 -->
    <property name="target" ref="target"/>
    <!-- 设置方法拦截器 -->
    <property name="interceptorNames" value="interceptor"/>
    <!-- 强制使用cglib动态代理 默认有接口就jdk 没接口就cglib -->
    <property name="optimize" value="true"/>
</bean>
```

Spring为我们提供的`ProxyFactoryBean`可以利用`目标类`+`方法拦截器`来生成`代理类` 我们只需要配置必要的参数即可... 不过这种代理有一个不好的地方就是我们需要通过`id=proxy`来获取到代理对象 过于繁琐 能不能有一种全自动的配置方式呢?



### SpringAOP编程

SpringAOP编程底层就是通过`BeanPostProcessor` 这样可以在Bean执行初始化方法的前后来生成代理对象 SpringAOP编程依赖于AspectJ提供的切入点表达式 可以从连接点当中选取切入点 所以说我们要导入依赖`spring-aspects` 它又会去依赖`aspectjweaver`的jar包. 而配置方式如下:

```xml
<!-- 配置目标类 -->
<bean class="me.faene.spring.bean.Target" id="target"/>
<!-- 配置切面类(包含若干通知) -->
<bean class="me.faene.spring.bean.Aspect" id="aspect"/>
<!-- 配置单个通知 -->
<bean class="me.faene.spring.bean.Advice" id="advice"/>

<!-- 代表这个区域是AOP的相关配置 -->
<aop:config>
    <!-- 定义切入点 使用AspectJ提供的切入点表达式 -->
    <aop:pointcut id="p1" expression="execution(* me.faene.spring.bean.Target.doit1())"/>
    <aop:pointcut id="p2" expression="execution(* me.faene.spring.bean.Target.doit2())"/>
    <!-- 定义切面 将切面类当中的若干通知织入到若干切入点 -->
    <aop:aspect ref="aspect">
        <aop:before method="advice1" pointcut-ref="p1"/>
        <aop:after method="advice2" pointcut-ref="p1"/>
    </aop:aspect>
    <!-- 将单个通知应用到一个切入点 -->
    <aop:advisor advice-ref="advice" pointcut-ref="p1"/>
</aop:config>
```



### 单个通知方式

如果想要配置单个通知 其实就是去实现`MethodInterceptor`接口 起初为了逻辑清晰我们说成是过滤器 实际上完全可以看做通知...

```java
public class Advice implements MethodInterceptor {
    @Override
    public Object invoke(MethodInvocation methodInvocation) throws Throwable {
        System.out.println("before");
        Object proxy = methodInvocation.proceed();
        System.out.println("after");
        return proxy;
    }
}
```

然后将单个通知应用到切入点即可:

```xml
<!-- 配置目标类 -->
<bean class="me.faene.spring.bean.Target" id="target"/>
<!-- 配置单个通知 -->
<bean class="me.faene.spring.bean.Advice" id="advice"/>
<!-- 将单个通知织入到切入点 -->
<aop:config>
	<aop:advisor advice-ref="advice" pointcut="execution(* *..*.*.doit2(..))"/>
</aop:config>
```



### 切面类方式

切面类方式要求我们提供一个切面类 切面类中按照格式来书写若干个方法(通知) 然后配置即可:

```java
public class Aspect {
    public void advice1() {
        System.out.println("Aspect.advice1");
    }
    public void advice2() {
        System.out.println("Aspect.advice2");
    }

```

```xml
<!-- 配置目标类 -->
<bean class="me.faene.spring.bean.Target" id="target"/>
<!-- 配置切面类(包含若干通知) -->
<bean class="me.faene.spring.bean.Aspect" id="aspect"/>
<aop:config>
    <!-- 定义切入点 使用AspectJ提供的切入点表达式 -->
    <aop:pointcut id="p1" expression="execution(* me.faene.spring.bean.Target.doit1())"/>
    <aop:pointcut id="p2" expression="execution(* me.faene.spring.bean.Target.doit2())"/>
    <!-- 定义切面 将切面类当中的若干通知织入到若干切入点 -->
    <aop:aspect ref="aspect">
        <aop:before method="advice1" pointcut-ref="p1"/>
        <aop:after method="advice2" pointcut-ref="p1"/>
    </aop:aspect>
</aop:config>
```



### 切入点表达式

语法: ==execution(修饰符 返回值 包名.类名.方法名(参数列表) throws 异常)==

修饰符: 一般省略, public, *

返回值: void, String, *

包: com.*.service 或者 com..service

参数列表: ..表示任意参数

方法名: *表示任意

异常: 可以省略

|| 代表多个  例如: execution(..) || execution(..) 并集



### AspectJ通知类型

```
try {
	before:前置通知
// 执行目标方法    
	afterReturning:返回通知
} catch (Exception e) {
	afterThrowing:异常通知
} finally {
	after:后置通知
}
around: 环绕通知
```

```java
// 注意包名 是aspectJ不是aop
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
public class Aspect {
	// 前置通知
    // <aop:before method="before" pointcut-ref="p2"/>
    public void before(JoinPoint joinpoint) {
        System.out.println("Aspect.before");
        System.out.println(joinpoint);
    }
	// 返回通知 可以获取到 目标方法的返回值
    // <aop:after-returning method="afterReturning" returning="result" pointcut-ref="p2"/>
    public void afterReturning(JoinPoint joinPoint, Object result) {
        System.out.println("Aspect.afterReturning");
        System.out.println(result);
    }
	// 异常通知 可以获取到 异常的相关信息
    // <aop:after-throwing method="afterThrowing" throwing="throwable" pointcut-ref="p2"/>
    public void afterThrowing(JoinPoint joinPoint, Throwable throwable) {
        System.out.println("Aspect.afterThrowing");
        System.out.println(throwable);
    }
	// 后置通知
    // <aop:after method="after" pointcut-ref="p2"/>
    public void after(JoinPoint joinPoint) {
        System.out.println("Aspect.after");
    }
	// 环绕通知 随心所欲
    // <aop:around method="around" pointcut-ref="p1"/>
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        Object result = null;
        try {
            System.out.println("before");
            result = joinPoint.proceed();
            System.out.println("afterReturing");
        } catch (Exception e) {
            System.out.println("afterThrowing");
        } finally {
            System.out.println("after");
        }
        return result;
    }
}
```



 

## Tx

### 编程式事务管理

基本思路: 业务层注入`TransactionTemplate`

```xml
~~ bean:deptDao
~~ bean:deptService
~~ bean:dataSource
~~ bean:transacitonManager

<bean class="org.springframework.transaction.support.TransactionTemplate" id="transactionTemplate">
    <property name="transactionManager" ref="transactionManager"/>
</bean>

<bean class="me.faene.service.DeptService" id="deptService">
    <property name="deptDao" ref="deptDao"/>
    <property name="transactionTemplate" ref="transactionTemplate"/>
</bean>
```

```java
public void doubleInsert(Dept d1, Dept d2) {
    this.transactionTemplate.execute(new TransactionCallbackWithoutResult() {
        @Override
        protected void doInTransactionWithoutResult(TransactionStatus transactionStatus) {
            deptDao.insert(d1);
            int i = 1 / 0;
            deptDao.insert(d2);
        }
    });
}
```

### FactoryBean事务管理

Spring事务管理本质上也是AOP嘛 我们通过`ProxyFactoryBean`可以创建代理对象 那么事务同样的会有`TransactionProxyFactoryBean`就可以帮助我们产生代理对象

```xml
~~ bean:deptDao
~~ bean:deptService
~~ bean:dataSource
~~ bean:transacitonManager

<bean id="proxyDeptService" 
      class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">
    <property name="target" ref="deptService"/>
    <property name="transactionManager" ref="transactionManager"/>
    <property name="transactionAttributes">
        <props>
            <!-- key: 哪些方法使用事务 可以使用通配符* -->
            <!-- text: 传播行为,隔离级别,是否只读 -->
            <prop key="*">PROPAGATION_REQUIRED,ISOLATION_DEFAULT,readOnly</prop>
        </props>
    </property>
</bean>
```

### 基于XML的AOP事务管理

```xml
~~ bean:deptDao
~~ bean:deptService
~~ bean:dataSource
~~ bean:transacitonManager

<!-- 通过<tx:advice>标签来产生一个方法拦截器(事务管理相关的) -->
<!-- id:看到id就是指BeanID 指定创建好的拦截器用什么id -->
<!-- transactionManager:创建肯定需要通过事务管理器 没什么好说的 -->
<tx:advice id="transactionInterceptor" transaction-manager="transactionManager">
    <!-- 具体的事务配置 -->
    <tx:attributes>
        <tx:method name="*" isolation="DEFAULT" propagation="REQUIRED" read-only="true"/>
    </tx:attributes>
</tx:advice>

<!-- 通过AOP配置 将事务拦截器 应用到具体的切入点上 -->
<aop:config>
    <aop:advisor advice-ref="transactionInterceptor" 
                 pointcut="execution(* me.faene.service.DeptService.doubleInsert(..))"/>
</aop:config>
```

### 基于注解的AOP事务管理

```xml
~~ bean:dataSource
~~ bean:transacitonManager

<tx:annotation-driven transaction-manager="transactionManager"
                proxy-target-class="true"/>  // 代表强制使用cglib代理
```

然后就只需要在业务层的具体实现类上添加`@Transactionl`注解即可

```java
//@Transactional 写在类上表示所有方法都采用这个配置
@Service
public class DeptService {

    @Autowired
    private DeptDao deptDao;

    @Transactional(isolation = Isolation.DEFAULT, propagation = Propagation.REQUIRED, readOnly = true)
    public void doubleInsert(Dept d1, Dept d2) {
        this.deptDao.insert(d1);
        int i = 1 / 0;
        this.deptDao.insert(d2);
    }
    
}
```













# Spring 注解



## IoC

### 组件扫描

注解IoC的基本原理就是 设置组件扫描 组件就是Bean的新称呼而已 然后通过注解来标注哪个类是组件就可以了

```xml
<context:component-scan base-package="me.faene.spring.bean" use-default-filters="false">
    // 如果只想包含某种注解的组件 就可以使用这种方式
    <context:include-filter type="annotation"
			expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
--------------
<context:component-scan base-package="me.faene.spring.bean">
    // 如果想要从包中排出某些注解的组件 就可以使用这种方式
    <context:exclude-filter type="annotation" 
            expression="org.springframework.stereotype.Component"/>
</context:component-scan>
```

那么配置好的组件扫描 Spring都会扫描哪些注解标注的组件呢?

* `@Component`: 用来说明这个类是一个Spring组件
* `@Controller`: 同上 不过有语义 指代三层架构中的web层组件
* `@Service`: 同上 不过有语义 指代三层架构中的业务层组件
* `@Repository`: 同上 不过有语义 指代三层架构当中的DAO层组件

上面的四种注解都可以通过默认的`value=""`属性来指定组件的ID



### 作用域

* `@Scope(value="singleton|prototype")`: 标注在组件类上



### 生命周期

* `@PostConstruct`: 标注在初始化方法上
* `@PreDestroy`: 标注在销毁方法上



## DI

* `@Value(value="")`: 可以注入普通类型的属性
* `@Autowired`: 可以根据类型来注入引用类型的属性
* `@Qualifer(value="beanID")`: 可以根据BeanID来注入引用类型的属性
* `@Resouce(name="beanID")`: 可以根据BeanID来注入引用类型的属性

上面这些注解可以写在很多地方:

* 写在属性上, ==不需要提供构造器和setter方法==

```java
@Component("person")
public class Person {
    @Value("小烯")
    private String name;
    @Value("20")
    private Integer age;
}
```

* 写在setter方法上

```java
@Component("person")
public class Person {
    private String name;
    private Integer age;
    @Value("小烯")
    public void setName(String name) { this.name = name; }
    @Value("20")
    public void setAge(Integer age) { this.age = age; }
}
```

* 在setter方法参数上

```java
public class Person {
    private String name;
    private Integer age;
    public void setName(@Value("abc") String name) {
        this.name = name;
    }
    public void setAge(@Value("20") Integer age) {
        this.age = age;
    }
}
```

* 在构造函数参数上

```java
@Component("person")
public class Person {
    private String name;
    private Integer age;
    public Person(@Value("abc") String name, @Value("20") Integer age) {
        this.name = name;
        this.age = age;
    }
}
```



## AOP

### 1. 核心配置文件

```xml
// 开启注解的IoC
<context:component-scan base-package="me.faene"/>
// 开启注解的AOP
<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```

### 2. @Aspect

这个注解用于标注切面类 当然目标类和切面类首先应该被`@Component`注解标注 只有成为了Spring组件才有AOP下去的意义

```java
@Component	// 说明这是一个Spring组件
@Aspect		// 说明这是一个切面类
public class UserServiceAspect {  // 某切面类
```

### 3. @Pointcut

这个注解用于定义公共的切入点 通过AspectJ切入点表达式 语法上如下:

```java
@Pointcut("execution(* *..*.*(..))")			// 使用切入点表达式定义全局切入点
private void myPointcut() {}					// 一个简单方法 私有 无返回值 无参数
```

如果想要引用这个切入点 就要:

```java
@Around("myPointcut()")			// 通过方法调用的语法来引用切入点 和execution()格式一致
public Object around(ProceedingJoinPoint joinPoint) throws Throwable {}
```

### 4. @Around..

`@Before` `@AfterReturing` `@AfterThrowing` `@After` `@Around` ............

```java
@Pointcut("execution(* *..*.*(..))")
private void myPointcut() {}
------------------------------------------
@Around("myPointcut()")
public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
    Object result = null;
    try {
    	// before
    	result = joinPoint.proceed();
    	// afterReturing
    } catch (Throwable t) {
    	// afterThrowing
    } finally {
    	// after
    }
    return result;
}
```

```java
------------------------------------------
@Before("execution(* *..*.*(..))")
public void before(JoinPoint joinPoint) {}
------------------------------------------
@AfterReturning(value = "myPointcut()", returning = "result")
public void afterReturing(JoinPoint joinPoint, Object result) {}
------------------------------------------
@AfterThrowing(value = "myPointcut()", throwing = "throwable")
public void afterTrowing(JoinPoint joinPoint, Throwable throwable) {}
------------------------------------------
@After("execution(* *..*.*(..))")
public void after(JoinPoint joinPoint) {}
```



# Spring整合



## Junit

```

```



## JdbcTemplate

基本思想: `XxxDao`需要使用`JdbcTemplate`

```xml
<bean class="com.mchange.v2.c3p0.ComboPooledDataSource" id="dataSource">
    <property name="driverClass" value="com.mysql.jdbc.Driver"/>
    <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/test"/>
    <property name="user" value="root"/>
    <property name="password" value="root"/>
</bean>

<bean class="org.springframework.jdbc.core.JdbcTemplate" id="jdbcTemplate">
    <property name="dataSource" ref="dataSource"/>
</bean>

<bean class="me.faene.dao.OrderDao" id="orderDao">
    <property name="jdbcTemplate" ref="jdbcTemplate"/>
</bean>
```

如果我们的`XxxDao`继承自`JdbcDaoSupport` 那么就可以直接注入数据源了

```xml
<context:property-placeholder location="classpath:db.properties"/>

<bean class="com.mchange.v2.c3p0.ComboPooledDataSource" id="dataSource">
    <property name="driverClass" value="${jdbc.driver}"/>
    <property name="jdbcUrl" value="${jdbc.url}"/>
    <property name="user" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>

<bean class="me.faene.dao.OrderDao" id="orderDao">
	<property name="dataSource" ref="dataSource"/>
</bean>
```

```java
public class OrderDao extends JdbcDaoSupport {
    public void doit() {
        super.getJdbcTemplate()...
    }
}
```



## Web/Servlet

思想: 在tomcat服务器启动的时候 将ApplicationContext容器放入到application域当中 

`web.xml`

```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
</context-param>
<!-- Spring整合Web 将Spring容器在服务器启动的时候放入到application域 -->
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

这样 我们只需要在Servlet当中 通过API来获取到application域当中的Spring容器对象即可

```java
public class MyServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) 
        	throws ServletException, IOException {
        // 在Servlet当中获取到Spring容器
        // 方式1
        ApplicationContext ioc1 = (ApplicationContext) this.getServletContext()
            .getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE);
        // 方式2
        ApplicationContext ioc2 =
            WebApplicationContextUtils.getWebApplicationContext(this.getServletContext());
    }
}
```

然后就可以获取到`XxxService`bean了 然后调用相关方法即可...



## Mybatis

### jar包

首先应该导入的是`mybatis` 和 `mybatis-spring`两个jar包....

### 传统的DAO开发

核心思想: 向Dao中注入`SqlSessionTemplate`对象....

```xml
<!-- SqlSessionFactory -->
<bean class="org.mybatis.spring.SqlSessionFactoryBean" id="sqlSessionFactory">
    <property name="dataSource" ref="dataSource"/>
    <!-- 指定Mybatis核心配置文件的位置 -->
    <property name="configLocation" value="classpath:mybatis/SqlMapConfig.xml"/>
</bean>

<!-- SqlSessionTemplate -->
<bean class="org.mybatis.spring.SqlSessionTemplate" id="sqlSessionTemplate">
    <constructor-arg index="0" ref="sqlSessionFactory"/>
</bean>

<bean class="me.faene.spring.dao.DeptDao" id="deptDao">
    <property name="sqlSessionTemplate" ref="sqlSessionTemplate"/>
</bean>
```

```java
public class DeptDao {
    private SqlSessionTemplate sqlSessionTemplate;
    public void setSqlSessionTemplate(SqlSessionTemplate sqlSessionTemplate) {
        this.sqlSessionTemplate = sqlSessionTemplate;
    }
}
```

当然 如果我们已经让我们的Dao继承了`SqlSessionDaoSupport`那么直接注入`SqlSessionFactory`即可...

```xml
<!-- SqlSessionFactory -->
<bean class="org.mybatis.spring.SqlSessionFactoryBean" id="sqlSessionFactory">
    <property name="dataSource" ref="dataSource"/>
    <!-- 指定Mybatis核心配置文件的位置 -->
    <property name="configLocation" value="classpath:mybatis/SqlMapConfig.xml"/>
</bean>

<!-- SqlSessionTemplate -->
<bean class="org.mybatis.spring.SqlSessionTemplate" id="sqlSessionTemplate">
    <constructor-arg index="0" ref="sqlSessionFactory"/>
</bean>

<bean class="me.faene.spring.dao.DeptDao" id="deptDao">
    <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
</bean>
```

```java
public class DeptDao extends SqlSessionDaoSupport {
}
```

### Mapper代理开发

核心思想: 让mybatis生成代理Mapper对象 放入到spring容器当中

```xml
<!-- SqlSessionFactory -->
<bean class="org.mybatis.spring.SqlSessionFactoryBean" id="sqlSessionFactory">
    <property name="dataSource" ref="dataSource"/>
    <!-- 指定Mybatis核心配置文件的位置 -->
    <property name="configLocation" value="classpath:mybatis/SqlMapConfig.xml"/>
</bean>

<!-- MapperFactoryBean来创建某个接口的代理对象 -->
<bean class="org.mybatis.spring.mapper.MapperFactoryBean" id="deptMapper">
    <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
    <property name="mapperInterface" value="me.faene.spring.mapper.DeptMapper"/>
</bean>
```

当然我们可以使用扫描器来批量扫描接口 然后创建代理Mapper:

```xml
<!-- SqlSessionFactory -->
<bean class="org.mybatis.spring.SqlSessionFactoryBean" id="sqlSessionFactory">
    <property name="dataSource" ref="dataSource"/>
    <!-- 指定Mybatis核心配置文件的位置 -->
    <property name="configLocation" value="classpath:mybatis/SqlMapConfig.xml"/>
</bean>

<!-- 配置扫描器 来自动扫描包 -->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    <property name="basePackage" value="me.faene.spring.mapper"/>
</bean>
```

MapperScannerConfigurer实现了`BeanDefinitionRegistryPostProcessor`接口所以说 可以向容器当中注册一堆Bean... 后置处理器意味着向容器填充的往往还是代理对象













