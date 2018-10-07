**spring**以简化企业级应用开发为己任。使用在

- Web 应用开发
- 数据库访问
- 应用集成
- 大数据处理

核心理念：IoC 与 AOP 。

**Spring MVC** 本身的发展也是如此：

- 最初的 XML 文件配置
- 后来的注解和 Java 代 码方式的配置
- 同时，Spring MVC 还在不断完善其测 试功能

**Spring Boot** 项目：这是一个改变游戏规则的项目，它能够

- 极大地简化配置
- 高效地管理依赖
- 并且与当下流行的微服务架构模式契合良好



第一个spring boot项目。

```
@SpringBootApplication
public class MasterspringmvcApplication {

    public static void main(String[] args) {
        SpringApplication.run(MasterspringmvcApplication.class, args);
    }
}
```

```
@SpringBootConfiguration--这个注解干嘛的？
@EnableAutoConfiguration--？？？
@ComponentScan--告诉 Spring 去哪里查找 Spring 组 件（服务、控制器等）。在默认情况下，这个注解将会扫描当前包以及该包下面的所有 子包。 
```



使用 Spring Boot 来编写 MVC 应用的第一步通常是在代码中**添加控制器**。将控制器放 到 controller 子包中，这样它就能够被@ComponentScan 注解所发现。

```
@Controller
public class HelloController {
    @RequestMapping("/")
    @ResponseBody
    public String hello() {
        return "hello,world";
    }
}
```



debug=true 

如果重新启动应用的话，就能看到Spring Boot 的自动配置报告。



### Spring Boot 帮助我们配置SpringMVC

Spring Boot 仅仅是基于常见的使用场景，帮助我们对应用进行配置。

- 分发器和 multipart 配置 

  DispatcherServletAutoConfiguration

- 视图解析器、静态资源以及区域配置 

   WebMvcAutoConfiguration，它声明了视图解析器、地域解 析器（localeresolver）以及静态资源的位置。

- 错误与转码配置

  ErrorMvcAutoConfiguration

- 嵌入式 Servlet 容器（Tomcat）的配置 

  EmbeddedServletContainerAutoConfiguration

- HTTP 端口

  application.properties 文件中定义 server.port 属性

- SSL 配置 

  server.ssl.key-store = classpath:keystore.jks server.ssl.key-store-password = secret server.ssl.key-password = another-secret

- 其它配置

  在 JacksonAutoConfiguration 中，声明使用 Jackson 进行 JSON 序列化； 

  在HttpMessageConvertersAutoConfiguration 中，声明了默认的HttpMessageConverter

  在 JmxAutoConfiguration 中，声明了 JMX 功能


## 精通MVC架构



自从在 Smalltalk 领域中提出这个理念，并在 Ruby on Rails 框架中采用之后，MVC 就 变得广受欢迎。

- 模型：包含了应用中所需的各种展现数据。 
- 视图：由数据的多种表述所组成，它将会展现给用户。 
- 控制器：将会处理用户的操作，它是连接模型和视图的桥梁。 

- 尽管 MVC 依然是当前设计 UI 的首选方案。
- 避免领域贫血的途径如下
- 服务层适合进行应用级别的抽象（如事务处理），而不是业务逻辑； 
- 领域对象应该始终处于合法的状态。通过校验器（validator）或 JSR-303 的校验注 解，让校验过程在表单对象中进行； 
- 将输入转换成有意义的领域对象； 
- 将数据层按照 Repository 的方式来实现，Repository 中会包含领域查询（例如参考 Spring Data 规范）； 
- 将领域逻辑与底层的持久化框架解耦； 
- 尽可能使用实际的对象，例如操作 FirstName 类而不是操作 String。 

**DDD 实践（领域驱动设计）**

实体（Entity）、值类型（value type）、通用语 言（Ubiquitous Language）、限界上下文（Bounded Context）、洋葱架构（Onion Architecture） 以及防腐化层（anti corruption layer）。。。



在Spring MVC 中，模型是由Spring MVC 的Model 或ModelAndView 封装的简单 Map。 它可以来源于数据库、文件、外部服务等，这取决于你如何获取数据并将其放到模型中。与数 据层进行交互的推荐方式是使用 Spring Data 库：Spring Data JPA、Spring Data MongoDB 等。 



Spring MVC 的控制层是通过使用@Controller 注解来进行处理的。在 Web 应用中，控 制器的角色是响应 HTTP 请求。带有@Controller 注解的类将会被 Spring 检索到，并且能够 有机会处理传入的请求。 



通过使用@RequestMapping 注解，控制器能够声明它们会根据 HTTP 方法（如 GET 或 POST 方法）和 URL 来处理特定的请求。控制器就可以确定是在 Web 响应中直接写入内容， 还是将应用路由一个视图并将属性注入到该视图中。 



纯粹的 RESTful 应用将会选择第一种方式，并且会在 HTTP 响应中直接暴露模型的JSON 或 XML 表述，这需要用到@ResponseBody 注解。在 Web 应用中，这种类型的架构 通常会与前端 JavaScript 框架关联，如 Backbone.js、AngularJS 或 React。在这种场景中， Spring 应用只需处理 MVC 中的模型层。



在第二种方式中，模型会传递到视图中，视图会由模板引擎进行渲染，并写入到响应 之中。 
视图通常会与某种模板方言关联，这种模板允许遍历模型中的内容，流行的模板方言 包括 JSP、FreeMarker 或 Thymeleaf。 



## 一个简单的例子

```
@Controller
public class HelloController {

    @RequestMapping("/")
    public String hello(Model model) {
        model.addAttribute("message","Hello from the controller");将该信息保存到模型中
        return "resultPage";
    }
}
```

```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head lang="en">
    <meta charset="UTF-8">
    <title>Hello thymeleaf</title>
</head>
<body>
    <span th:text="${message}">Hello html</span>展现来自模型中的信息
</body>
</html>
```



**Spring 表达式语言**

当使用“${}”语法时，我们实际上使用的是 Spring 表达式语言（Spring Expression Language，SpEL）。关于 EL，有多个不同的变种，而 SpEL 是其中威力强大的一种。 

访问列表元素 list[0] 

访问 Map 条目 map[key] 



- 在视图中展现来自服务端的数据：Model model
- 获取用户的输入：@RequestParam("name") String userName





## REST

REST（表述性状态转移，Representational State Transfer）是一种架构风格，它定义了 创建可扩展 Web 服务的最佳实践，这个过程中会充分发挥 HTTP 协议的功能。 

- 客户端-服务器：UI是与数据存储分离的。 

- 无状态：每个请求会包含服务器所需的足够信息，无需维护状态就能进行操作。 
- 可缓存：服务器的响应中包含了足够的信息，客户端能够对数据存储做出合理的 决策。
- 统一接口：URL 会唯一识别资源，能够通过超链接发现 API。
- 分层：API 的每个资源都提供了都合理程度的细节。 



这种架构的优势在于易于维护以及便于进行服务发现。它的扩展性也很好，因为没有 必要在服务器和客户端之间维护持久化的连接，这样就没有必要进行负载均衡或会话粘性。 最后，因为服务的内容非常简洁且易于缓存，所以服务会更加高效。



第 0 级——HTTP 

第 1 级——资源 

第 2 级——HTTP 动作 

第 3 级——超媒体控制 ？？













