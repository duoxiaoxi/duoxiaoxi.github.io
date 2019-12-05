# Spring MVC





## 组件介绍

`前端控制器(DispatcherServlet)`: 请求过来首先访问的就是前端控制器

`处理器映射器(HandlerMapping)`: 前端控制器会通过处理器映射器来查找 查找结果是:

`处理器执行链(HandlerExecutionChain)`: 里面包含了处理器和一系列的拦截器

`处理器适配器(HandlerAdapter)`: 得到了处理器 就需要找这个处理器适配的适配器 这样适配器才能去执行他

`ModelAndView`: 处理器适配器执行处理器的结果就是ModelAndView

`视图解析器(ViewResolver)`: 视图解析器可以真正的将ModelAndView解析为View



### 源码跟踪

当我们的请求到达`DispatcherServlet`就会去调用到`doDispatch()`方法:

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
	// 通过自己的方法getHandler()来尝试通过request获取到处理器执行链(包含了处理器)
    HandlerExecutionChain mappedHandler = this.getHandler(request);
    // 通过自己的方法getHandlerAdapeter()来尝试获取可以适配找到的处理器的处理器适配器
    HandlerAdapter ha = this.getHandlerAdapter(mappedHandler.getHandler());
    // 通过刚到找到的处理器适配器 让它去执行我们的处理器... 得到模型&视图
    ModelAndView mv = ha.handle(request, response, mappedHandler.getHandler());
}
```

`this.getHandler(HttpServletRequest request)`

```java
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    	// 依次遍历所有的映射器 尝试她们获取到处理器执行链 一旦某个获取成功 立即返回
		for (HandlerMapping hm : this.handlerMappings) {
			HandlerExecutionChain handler = hm.getHandler(request);
			if (handler != null) {
				return handler;
			}
		}
		return null;
	}
}
```

`this.getHandlerAdapeter(Object handler)`

```java
protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
	// 依次遍历所有的处理器适配器 调用supports()方法判断是否适配指定的处理器 一旦适配立即返回
	for (HandlerAdapter ha : this.handlerAdapters) {
		if (ha.supports(handler)) {
			return ha;
		}
	}
	throw new ServletException("...");		// 没有找到就抛出异常
}
```







### 前端控制器

```xml
<!-- 配置前端控制器 所有的请求都会经过这个Servlet 而这个Servlet会加载SpringMVC的配置文件 -->
<servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

默认加载的配置文件是 `/WEB-INF/ServletName-servlet.xml` 所以我们需要通过一个初始化参数来指定配置文件的位置...

> 路径问题 DispatcherServlet 的路径必须配置成`/` 而不是==/*== /*会解析jsp 这是8彳亍的



### 处理器映射器

`BeanNameUrlHandlerMapping`: 通过bean的name来进行映射 根据url找对应的name对应的bean

```xml
<bean class="me.faene.controller.TestController" id="/test"/>

<!-- 处理器映射器 -->
<!-- 顾名思义 就是将bean的id/name作为url进行查找 -->
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
```

`SimpleUrlHandlerMapping`: 通过简单的配置 将url映射到对应的bean(也就是我们书写的处理器)

```xml
<bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
    <property name="mappings">
        <props>
            <!-- key:servletPath value:BeanID -->
            <prop key="/page1">testController</prop>
            <prop key="/page2">testController</prop>
            <prop key="/page3">testController</prop>
        </props>
    </property>
</bean>
```



### 处理器适配器


`HttpRequestHandlerAdaper`: 支持处理"实现了HttpRequestHandler接口"的处理器

```xml
<bean class="org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter"/>
```

```java
public class Test2Controller implements HttpRequestHandler {
    @Override
    public void handleRequest(HttpServletRequest httpServletRequest,
			HttpServletResponse httpServletResponse) throws ServletException, IOException {
    }
}
```

`SimpleControllerHandlerAdapter`: 支持处理"实现了Controller接口"的处理器

```xml
<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>
```

```java
public class TestController implements Controller {
    @Override
    public ModelAndView handleRequest(
        HttpServletRequest request, HttpServletResponse response) throws Exception {
    }
}
```




### 默认的HM&HA

如果我们不在配置文件中配置处理器映射器和处理器适配器 找到如下配置文件:

`spring-webmvc.jar\org\springframework\web\servlet\DispatcherServlet.properties`

```properties
HandlerMapping=org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping,\
	org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping

HandlerAdapter=org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter,\
	org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter,\
	org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter
	
ViewResolver=org.springframework.web.servlet.view.InternalResourceViewResolver
```

在Spring3.1之前 使用的是==DefaultAnnotationHandlerMapping==+==AnnotationMethodHandlerAdapter==作为注解的处理器映射器和处理器适配器 不过在Spring3.1之后已经使用了别的了... 所以说我们肯定要用新的==RequestMappingHandlerMapping==+==RequestMappingHandlerAdapter== 以及使用==<mvc:annotation-driven&gt;==可以代替配置Bean 以及加载了很多参数绑定方法



### 视图解析器 

对于我们的处理器方法(HandlerMethod)的返回值String类型而言 代表了请求转发或者重定向的路径 具体有如下几种写法:

* `abc`: 这种写法最终会进行拼接 ==> `prefix`+`abc`+`suffix`
* `forward:abc`: 这种不以`/`开头的路径是相对于当前HandlerMethod所在的虚拟路径 也就是@RequestMapping中书写的路径(包括标注在类上的根路径)
* `forward:/abc`: 这种以`/`开头的路径是相对于服务器路径的根路径 也就是说我们可以直接索引到/index.jsp
* `redirect:abc` `redirect:/abc`: 与上面两种一致 只不过是重定向 注意不要重定向到WEB-INF文件夹内部 并且这里书写的也是**服务器路径** 而不是浏览器路径... 需要注意

```xml
<!-- 视图解析器 -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/pages/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

```java
/*
 - ctrls
 - - ctrl1
 - - ctrl2
 - - ctrl3
 - WEB-INF
 - - pages
 - - - page1.jsp
 - - - page2.jsp
 - index.jsp
 */
@Controller
@RequestMapping("/ctrls")
public class TestController {

    @RequestMapping("/ctrl1")
    public String ctrl1() {
        // prefix + path + suffix =
        // /WEB-INF/pages/ + ../pages/page1 + .jsp
        return "../pages/page1";
    }

    @RequestMapping("/ctrl2")
    public String ctrl2() {
        // ServletPath:/WEB-INF/pages/page1.jsp
        return "forward:/WEB-INF/pages/page2.jsp";
    }

    @RequestMapping("/ctrl3")
    public String ctrl3() {
        // 当前路径: /ctrls
        // 相对路径: ../index.jsp
        return "redirect:../index.jsp";
    }
}
```



### 注解开发

想要进行注解开发 肯定要去配置注解的处理器映射器和注解的处理器适配器 有两种方法:

```xml
<!-- 1. 配置注解的处理器映射器和处理器适配器 -->
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/>
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"/>
```

```xml
<!-- 2. 相当于配置了上面的两个 并且有新东西 -->
<mvc:annotation-driven></mvc:annotation-driven>
```

那么 注解的处理器映射器是如何映射的呢? 注解的处理器适配器又是怎么处理的呢?

* 注解的处理器<u>映射器</u>会去SpringMVC容器当中检查被`@Controller`注解标注的类 再查找这些类中被`@RequestMapping`标注的方法 这个方法就是处理器了
* 注解的处理器<u>适配器</u>会执行找到的方法 无论通过什么方法...

需要注意的试点是==去容器中寻找被@Controller注解标注的类== 首先要保证在容器中 才会去找 这意味着我们可以:

```xml
<bean class="me.faene.controller.MyController"/>
```

通过配置文件来保证我们的Controller在容器中 但是既然已经被@Controller注解标注了 为什么不用组件扫描呢?

```xml
<context:component-scan base-package="me.faene.controller" use-default-filters="false">
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```



## 常用注解

### @RequestMapping

这个注解用来确定什么样的请求可以访问到这个处理器 都是通过这个注解的属性来实现的:

* `value`: 指定访问处理器的ServletPath 支持Ant通配符

> Ant风格的通配符如下:
>
> * `?`: 匹配一个字符
> * `*`: 匹配任意个字符
> * `**`: 匹配多层路径

* `method:RequestMethod[]`: 用来指定访问方式 可以指定多种
* `params:String[]`: 用来指定请求参数 支持基本表达式
* `headers:String[]`: 用来指定请求头 支持基本表达式

> 基本表达式如下:
>
> * `key`: 必须含有指定key的参数或者头
> * `!key`: 不能包含指定key的参数或者头
> * `key=value`: 必须包含指定key的参数或者头 并且属性必须为value
> * `key!=value`: 包含指定key的单数或者头 并且属性不能为value    或者    不包含

示例代码:

```java
@RequestMapping(
        value = "/test/**/test",
        method = {RequestMethod.GET, RequestMethod.POST},
        params = {"a", "!b", "c=C", "d!=D"},
        headers = {"Host=localhost:8080"}
)
public ModelAndView test() {
    return new ModelAndView("index")
            .addObject("var", "abc");
}
```

另外这个注解可以标注在类上或者方法上 如果标注在了类上 表示这个类的所有处理器访问时都要采取指定的前缀:

```java
@Controller
@RequestMapping("/test")
public class MyController {
    @RequestMapping(value = "/test1")		// 浏览器访问路径: /test/test1
    public ModelAndView test() {
        return null;
    }
}
```

这样 我们浏览器请求的ServletPath必须是**/test/test1**



### @PathVariable

这个注解可以获取到我们请求的ServletPath当中的相关信息 例如:

```java
@RequestMapping("/test/{var1}/{var2}")
public ModelAndView test(
        @PathVariable("var1") Integer var1,
        @PathVariable("var2") String var2
) { return new ModelAndView("index"); }
```



### @RequestParam

这个注解可以将我们的请求参数获取到处理器方法的参数当中:

* `value`: 指定请求参数的名称
* `required:boolean`: 指定是否必须提供该请求参数
* `defaultValue:String`: 指定默认值

示例代码:

```java
@RequestMapping("/test")
public ModelAndView test(
        @RequestParam(value = "username", required = true) String username,
        @RequestParam(value = "password", required = false, defaultValue = "12345") String password
) {
    System.out.println("username = [" + username + "], password = [" + password + "]");
    return null;
}
```



### @RequestHeader

同理我们可以通过上面的注解来获取请求头 这个注解的三个属性和@RequestParam也一致



### @CookieValue

同理我们可以通过上面的注解来获取Cookie 这个注解的三个属性和@RequestParam也一致



### ServletAPI获取

如果我们的处理器 也就是Controller当中的方法的参数没有被上面讲解的几个注解修饰 也就是无注解修饰 往往他们都有如下的几个含义 根据他们的参数类型进行分类:

* `HttpServletRequest`
* `HttpServletResponse`
* `HttpSession`
* `InputStream` `OutputStream` `Reader` `Writer`

也就是说 我们可以通过参数绑定来获取到ServletAPI中的常用对象...

当然还有常见的参数绑定 也就是书写我们的POJO类作为参数 然后表单项采用OGNL表达式即可....



### 乱码问题

我们需要配置一个过滤器来解决POST请求乱码的问题:

```xml
<filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

==这个过滤器必须放在所有过滤器的最前面 否则无效==



### 模型数据

当业务层进行处理得到结果之后 我们需要做两件事情 将一些数据保存到模型当中 然后进行请求转发... 因此我们的处理器方法就有了几种写法:

* 返回ModelAndView

```java
@RequestMapping("/test1")
public ModelAndView test1() {
    ModelAndView mv = new ModelAndView();
    mv.setViewName("show");					// 请求转发
    mv.addObject("obj", new Object());		// 封装模型
    return mv;
}
```

* 返回String 参数中使用Map或者Model或者ModelMap

```java
@RequestMapping("/test2")
public String test2(Map<String, Object> model) {
    model.put("obj", new Object());
    return "show";
}

@RequestMapping("/test3")
public String test3(Model model) {
    model.addAttribute("obj", new Object());
    return "show";
}

@RequestMapping("/test4")
public String test4(ModelMap model) {
    model.addAttribute("obj", new Object());
    return "show";
}
```

这种方式也可以进行 重定向 只需要在返回的字符串中使用前缀: `redirect:path` 如果我们打印`Map` `Model` `ModelMap`的实际类型 会发现他们的实际类型其实都是`BindingAwareModelMap`



### @SessionAttribute

我们向ModelMap或者ModelAndView中存入的变量 都是request域的 那么如何才能存储到session域呢?

```java
@Controller
@SessionAttributes(
        names = {"dept"},       // 如果模型中有名为dept的变量 就将其拷贝到session域
        types = {Dept.class}    // 如果模型中有类型为Dept的变量 就将其拷贝到session域
)
public class DeptController {
    @RequestMapping(value = "/dept", method = RequestMethod.POST)
    public ModelAndView test(Dept dept) throws Exception {
        return new ModelAndView("show")
                .addObject("dept", dept);
    }
}
```



### @ModelAttribute

这个注解可以放在方法上 或者方法参数上 如果标注在方法上 那么这个方法会在@RequestMapping标注的方法之前执行 并且总会向模型Model中填充一些对象... 如果标注在方法参数上 则会从模型中获取一些对象 并且还会被请求参数进行参数绑定...

* 标注在方法上, 方法没有返回值, 参数有Model

```java
// 模型中的数据: {obj:new Ojbect()}
@ModelAttribute
public void before1(Model model) {
    model.addAttribute("obj", new Object());
}
```

* 标注在方法上, 方法返回对象类型, 没有参数

```java
// 模型中的数据: {object:new Object()} 键是类型的首字母小写
@ModelAttribute
public Object before2() {
    return new Object();
}
```

* 标注在方法上, 方法返回对象类型, 没有参数, 但是注解提供了参数

```java
// 模型中的数据: {obj:new Object()}
@ModelAttribute("obj")
public Object before3() {
    return new Object();
}
```

* 标注在参数上 注解提供了参数

```java
// 代表从BindingAwareModelMap中获取key=dept的对象 然后在进行参数绑定
@RequestMapping("/dept/{id}")
public String update(@ModelAttribute("dept") Dept dept) {
    System.out.println("dept = " + dept);
    return "show";
}
```



### HttpSessionRequiredException

对于@ModelAttribute和@SessionAttribute会引发一个异常HttpSessionRequiredException 我们先考虑一个问题:        方法的参数`Dept dept`和`@ModelAttribute Dept dept`的区别是什么?

答: 前者只进行参数绑定 后者先尝试从模型对象中获取 **再尝试从SessionAttribute中获取** 最终参数绑定

因为问题的原因就在于 还要尝试从SessionAttribute中获取 因为这是你承诺的东西....

```java
@Controller
@SessionAttributes("dept_1")
public class DeptController {

    @ModelAttribute("dept_0")       // 向隐藏模型中存储 dept_0
    public Dept save() {
        return new Dept();
    }

    @RequestMapping("/get1")
    public String get1(Dept dept) { // 创建新对象 进行参数绑定
        return "show";
    }

    // 下面这种情况 首先尝试从隐藏模型中获取 dept_1 发现没有
    // 又从你承诺的SessionAttribute中尝试获取 dept_1
    // 然而Session域中实际并没有这个对象 所以就会抛出异常了...
    @RequestMapping("/get2")
    public String get2(@ModelAttribute("dept_1") Dept dept) {
        return "show";
    }

    // 如果不指定value属性 那么默认就是去找 dept, 隐藏模型没有, session也没有
    // 所以会自己创建一个新的 所以不会报错...
    @RequestMapping("/get3")
    public String get3(@ModelAttribute Dept dept) {
        return "show";
    }
    
}
```



### 处理器总结

```java
@Controller
@RequestMapping("/base")
@SessionAttributes(
        names = {"dept"},
        types = {Dept.class}
)
public class DeptController {
    
    @ModelAttribute("dept")
    public Dept before() {
        return new Dept();
    }
    
    @RequestMapping(
            value = "/dept/*/{ cid}",
            method = {RequestMethod.GET, RequestMethod.PUT},
            headers = {"User-Agent!=..."},
            params = {"a", "!b", "c=C", "d!=D"}
    )
    public String service(
            @PathVariable("id") Integer id,             // 获取URI信息
            @RequestParam("name") String name,          // 获取请求参数
            @RequestHeader("Host") String host,         // 获取请求头
            @CookieValue("JSESSIONID") String jid,      // 获取Cookie
            @ModelAttribute("dept") Dept dept1,         // 对隐藏模型参数绑定
            Dept dept2,                                 // 参数绑定
            HttpServletResponse response,               // 获取原生API
            Model model                                 // 获取隐藏模型
    ) {
        return "show";
    }
    
}
```



## 配置文件

### &lt;mvc:default-servlet-handler&gt;

我们所有的请求都会经过`DispatcherServlet`, 因为它的`<url-pattern>`被我们配置成了`/`, 如果配置成`.action`/`.do`的话, 就不用担心静态资源例如`/js/jquery.js`被拦截了.. 但是==优雅的Restful风格URL是不建议我们不应该带有后缀名==这个时候我们就需要来解决静态资源映射的问题, 只需要配置如下的标签:

```xml
<!-- 静态资源配置 -->
<mvc:default-servlet-handler></mvc:default-servlet-handler>
```

那么这个配置的作用我们已经猜到了:

* 如果请求可以找到对应的Handler ==> 执行Handler
* 如果请求找不到对应的Handler ==> 去查找静态资源, 如果没找到`返回404`, 而不是`没有提供映射`

那么原理是什么呢?

在我们的tomcat服务器配置文件`tomcat:conf/web.xml`中有着对所有项目的`公共部署描述符`配置, 里面配置了如下内容:

```xml
<servlet>
    <servlet-name>default</servlet-name>
    <servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
    ... initParams
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>default</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

这个配置文件中的两个Servlet我们其实已经学习过了, 一个用来处理静态资源, 一个用来处理JSP页面...

如果我们配置了`<mvc:default-servlet-handler>` 那么就会向容器中添加一个`Handler`:

```java
public class DefaultServletHttpRequestHandler implements HttpRequestHandler {
	@Override
	public void handleRequest(HttpServletRequest request, HttpServletResponse response) {
		RequestDispatcher rd = 
            	this.servletContext.getNamedDispatcher(this.defaultServletName);
		rd.forward(request, response);
	}
}
```

也就是说, 我们的请求会被`HandlerMapping`尝试进行映射:

* 如果映射到我们编写的Handler, 就该怎么样怎么样
* 如果没有映射到我们编写的Handler, 就会查看容器中是否有`DefaultServletHttpRequestHandler`
  * 如果有, 就映射到它, 它就会请求转发到`DefaultServlet`, 就会处理静态资源了
  * 如果没有, 就发送异常, 说明当前请求没有映射



### &lt;mvc:view-controller&gt;

有的时候我们我们可能会写出如下的控制器...

```java
@RequestMapping("/index.html")
public String goIndex() {
    return "index";				// [/WEB-INF/jsp/index.jsp]
}
```

实际上我们这个处理器方法存在的意义只是转发到一个视图而已... 很明显这是一个相当固定的内容... 我们应该将其放入到配置文件中才合适:

```xml
<mvc:view-controller path="/index.html" view-name="index"/>
```

这就是这个标签存在的意义, 将一个`ServletPath`请求映射到一个`SpringMVC视图名称`



### &lt;mvc:annotaion-driven&gt;








## JSON

**Jackson**

如果我们想要在我们的SpringMVC项目当中使用Json的相关功能, 需要导入Jackson的jar包:

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.8</version>
</dependency>
```

`jackson-databind` <== `jackson-core` + `jackson-annotations` 我们只需要导入上面的jar包即可



**@RequestBody**





**@ResponseBody**







## 上传

如果我需要进行上传相关的操作, 我们需要引入如下的jar包 `commons-fileupload` <= `commons-io`:

```xml
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.1</version>
</dependency>
```

然后需要在`spring.mvc`中配置一个`CommonsMultipartResolver`:

```xml
<bean  id="multipartResolver"	# 必须写这个ID: multipartResolver
      	class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <property name="defaultEncoding" value="UTF-8"/>
    <property name="maxUploadSize" value="1048576"/>
    <property name="maxUploadSizePerFile" value="1048576"/>
</bean>
```

* `defaultEncoding`: 设置对客户端传来的文件的解码方式
* `maxUploadSize`: 设置最大的上传文件总大小
* `maxUploadSizePreFile`: 设置上传时单个文件的最大SIZE
* ==`id=multipartResolver`: ID必须要用这个, 否则SpringMVC是无法理解的...==

然后我们就可以在我们的控制器方法中通过`@RequestParam`注解轻松的获取到文件了:

```html
<form action="/web/testUpload" method="post" enctype="multipart/form-data">
    <input type="file" name="file">
    <button>testUpload</button>
</form>
```

```java
@RequestMapping("/testUpload")
public String testUpload (
        @RequestParam("file") MultipartFile file
) throws Exception {
    System.out.println(file);
    return "test";
}
```

对于上传的文件我们需要用`MultipartFile`类型来进行接收, 这个类有如下方法:

* `getName():String`: 获取`<input name="?">`中的`?`, 也就是**请求参数名称**
* `getOriginalFileName():String`: 获取上传文件的**文件名**
* `getSize():long`: 获取上传文件的**大小**
* `getContentType():String`: 获取上传文件的**MEME类型**
* `getBytes():byte[]`: 获取到文件的二进制流**byte数组**
* `isEmpty():boolean`: 判断接收的文件是否为空
* `getInputStream():InputStream`: 获取到文件的**输入流**
* `transferTo(File):void`: 将接收到的文件拷贝到一个新的路径...



## 拦截器

### 自定义拦截器

* 第一步: 创建拦截器类, 实现`HandlerInterceptor`接口

```java
public class MyInterceptor implements HandlerInterceptor {
    @Override		// 此方法在处理器执行前执行
    public boolean preHandle(...) throws Exception { return true; }
    @Override		// 此方法在处理器执行后执行
    public void postHandle(...) throws Exception {}
    @Override		// 此方法在视图渲染完成后执行, 一般用作资源释放
    public void afterCompletion(...) throws Exception {}
}
```

注意第一个方法`preHandler()`应该返回`true`, 稍微我们会说明为什么...

* 第二步: 在`springmvc.xml`当中进行配置

```xml
<!-- 自定义拦截器 -->
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/dept/test"/>
        <!--<mvc:exclude-mapping path=""/>: 用来排除一些路径-->
        <bean class="me.faene.interceptor.MyInterceptor"/>
        <!--<ref bean=""/>: 用来引用一个Bean-->
    </mvc:interceptor>
</mvc:interceptors>
```

这个配置很简单, 我们就不再赘述了... 都是我们非常熟悉的东西了

> 如果我们的拦截器要拦截所有的处理器, 那么就没必要使用`<mvc:mapping>`标签了, 这个时候我们甚至可以直接这样配置:
>
> ```xml
> <mvc:interceptors>
>     <bean class="me.faene.interceptor.MyInterceptor"/>
>     <!--<ref bean=""/>-->
> </mvc:interceptors>
> ```



### 拦截器源码

我们还是要从`DispatcherServlet#dispatcher()`方法说起... 

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) {
    // ...... 处理器适配器获取完毕 准备处理
    if (!mappedHandler.applyPreHandle(processedRequest, response)) {
        return;
    }
    mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
    mappedHandler.applyPostHandle(processedRequest, response, mv);
    // ...... 即将视图渲染...
    processDispatchResult(processedRequest, response, mappedHandler, mv);
}
```

---

这里的`mappedHandler`变量就是我们的`HandlerExecutionChain`, 内部包含了`方法处理器`以及若干`拦截器`. 首先会调用`applyPreHandle()`方法:

```java
boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) {
    HandlerInterceptor[] interceptors = getInterceptors();	// 获取所有的拦截器
    for (int i = 0; i < interceptors.length; i++) {
        HandlerInterceptor interceptor = interceptors[i];
        // 依次执行每个拦截器的preHandler()方法, 只要有一个返回FALSE 立即返回FALSE
        if (!interceptor.preHandle(request, response, this.handler)) {
            triggerAfterCompletion(request, response, null);	// 后期说明
            return false;
        }
        this.interceptorIndex = i;		// 将一个变量interceptorIndex设置为已经执行的拦截器数
    }
    return true;
}
```

需要注意一点, 如果有一个拦截器的`preHandler()`方法返回了`FALSE`, 那么`applyPreHandler()`方法返回`FALSE`, 从而导致`doDispatcher()`方法直接`return;`退出方法... 就结束了

---

紧接着就是处理器适配器开始处理我们的`处理器方法`了, 得到`ModelAndView`:

```java
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
```

---

再之后开始调用`applyPostHandle()`方法, 我们可以猜到就是一次执行每个拦截器的`postHandler()`方法:

```java
void applyPostHandle(HttpServletRequest request, HttpServletResponse response, ModelAndView mv) {
    HandlerInterceptor[] interceptors = getInterceptors();
    for (int i = interceptors.length - 1; i >= 0; i--) {
        HandlerInterceptor interceptor = interceptors[i];
        interceptor.postHandle(request, response, this.handler, mv);
    }
}
```

---

紧接着就要去调用渲染视图的方法了,

```java
private void processDispatchResult(...) throws Exception {
	// ...... 若干操作
    if (mv != null && !mv.wasCleared()) {
        render(mv, request, response);		// 调用渲染视图方法
    }
	// ...... 若干操作
    if (mappedHandler != null) {
        mappedHandler.triggerAfterCompletion(request, response, null);
    }
}
```

---

视图渲染完成之后, 紧接着就调用了`triggerAfterCompletion()`方法, 显然就是依次调用每个拦截器的`afterCompletion()`方法...

```java
void triggerAfterCompletion(...) throws Exception {
    HandlerInterceptor[] interceptors = getInterceptors();
    for (int i = this.interceptorIndex; i >= 0; i--) {		// 注意这里是interceptorIndex
        HandlerInterceptor interceptor = interceptors[i];
		interceptor.afterCompletion(request, response, this.handler, ex);
    }
}
```



### 拦截器源码总结

经过我们的源码分析 我们可以发现`HanderExecutionChain`有如下的三个方法:

* `applyPreHandler() {int i = 0; i < interceptors.length; i++}`
  * `正序` 依次调用每个拦截器的 `preHandler()`方法
  * 记录一个`interceptorIndex`的变量值, 代表已经处理完成`preHandler()`方法的拦截器个数...
  * 一旦有拦截器`preHandler()`方法返回`FALSE`, 立即调用`tiggerAfterComletion()`方法, 然后`结束本次请求处理(return;)`
* `applyPostHandler() {for (int i = interceptors.length - 1; i >= 0; i--)}`
  * `逆序` 依次调用每个拦截器的`postHandler()`方法
* `triggerAfterCompletion() {for (int i = this.interceptorIndex; i >= 0; i--)}`
  * `逆序` 依次调用==已经执行完`preHandler()`方法的拦截器==的`afterCompletion()`方法



而`HandlerExecutionChain`的执行时机如下:

1. 遍历处理器映射器获取到`HandlerExecutionChain`
2. 遍历处理器适配器找到合适的适配器
3. ==调用处理器执行链的`applyPreHandler()`方法==
   1. 如果没有拦截器`preHandler()`方法返回`FALSE`, 执行{4}
   2. 如果有, 执行{8}
4. 处理器适配器开始处理... 正在执行处理器方法
5. ==调用处理器执行链的`applyPostHandler()`方法==
6. 遍历视图解析器获取到`ModelAndView`
7. 解析视图得到`View`, 调用其`render()`渲染方法
8. ==调用处理器执行链的`tiggerAfterCompletion()`方法==



### 拦截器执行顺序

我们分析完拦截器的源码, 这个执行顺序简直已经一目了然了, 现在假设我们有三个拦截器: `Icpt1` `Icpt2` `Icpt3`, 他们都实现了`Pre` `Post` `After` 三个方法, 在不同的拦截器`Pre`方法返回`FALSE`的情况下, 执行顺序是什么样子的呢?

* **没有返回FALSE的**: `1Pre` `2Pre` `3Pre` `HA` `3Post` `2Post` `1Post` `3After` `2After` `1After`


* **Icpt1-Pre返回FALSE**: `1Pre`
* **Icpt2-Pre返回FALSE**: `1Pre` `2Pre` `1After` 
* **Icpt3-Pre返回FALSE**:  `1Pre` `2Pre` `3Pre` `2After` `1After`

我们可以简化我们记忆的思路了, 其实只要记住如下的两点就好了:

* `Pre` `Post` `After` 三个方法总是 `正` `逆` `正` 的顺序去执行
* 已经执行完的`Pre`方法一定要通过对应的`After`来==释放资源==




## Restful风格

| 请求方式 |   请求路径   |      实际含义      |
| :------: | :----------: | :----------------: |
|   POST   |   /domain    |    新增一个实体    |
|  DELETE  | /domain/{id} | 根据ID删除一个实体 |
|   PUT    | /domain/{id} | 根据ID更新一个实体 |
|   GET    | /domain/{id} | 根据ID获取一个实体 |
|   GET    |   /domains   |   获取所有的实体   |

上面的ServletPath形式就是我们所说的Restful风格.........



### HiddenHttpMethodFilter

浏览器的form表单只支持POST和GET 而DELETE和PUT并不支持 所以说Spring提供了一个过滤器 可以帮助我们将请求方式变为4种标准形式 首先我们应该在`web.xml`当中进行配置:

```xml
<filter>
    <filter-name>hiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>hiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

这个过滤器执行的原理就是根据请求参数中`_method`的值将POST请求转换为指定的请求

```jsp
<form action="${pageContext.request.contextPath}/domain/1" method="post">
    <input type="hidden" name="_method" value="GET">
    <button>Submit! GET</button>
</form>

<form action="${pageContext.request.contextPath}/domain" method="post">
    <input type="hidden" name="_method" value="POST">
    <button>Submit! POST</button>
</form>

<form action="${pageContext.request.contextPath}/domain/1" method="post">
    <input type="hidden" name="_method" value="PUT">
    <button>Submit! PUT</button>
</form>

<form action="${pageContext.request.contextPath}/domain/1" method="post">
    <input type="hidden" name="_method" value="DELETE">
    <button>Submit! DELETE</button>
</form>
```

而在控制器当中 我们就可以根据不同的请求方式来访问到不同的处理器:

```java
// 新增一个实体
@RequestMapping(value = "/domain", method = RequestMethod.POST)
public ModelAndView add() {
    return null;
}

// 更新一个实体
@RequestMapping(value = "/domain/{id}", method = RequestMethod.PUT)
public ModelAndView update(@PathVariable("id") Integer id) {
    return null;
}

// 删除一个实体
@RequestMapping(value = "/domain/{id}", method = RequestMethod.DELETE)
public ModelAndView delete(@PathVariable("id") Integer id) {
    return null;
}

// 查找一个实体
@RequestMapping(value = "/domain/{id}", method = RequestMethod.GET)
public ModelAndView find(@PathVariable("id") Integer id) {
    return null;
}
```



## 国际化





