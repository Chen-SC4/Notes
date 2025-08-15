# Spring

## 1. 循环依赖

循环依赖是指两个或多个 Bean 之间相互依赖的情况。例如，Bean A 依赖于 Bean B，而 Bean B 又依赖于 Bean A。

```java
@Service
public class A {
	@Autowired
	private B b;
}

@Service
public class B {
	@Autowired
	private A a;
}
```

在 Spring 中，默认情况下，Bean 的创建是按顺序进行的。当 Spring 尝试创建 Bean A 时，发现它依赖于 Bean B，就会暂停 Bean A 的创建，转而去创建 Bean B。在创建 Bean B 时，又发现它依赖于 Bean A，此时 Spring 就会陷入一个无限循环，无法继续创建 Bean。

### 如何解决循环依赖

为了解决**循环依赖**的问题，Spring 采用了**三级缓存**机制。在 Bean 的创建过程中，将正在创建的 Bean 的**不同阶段的状态**放入不同的缓存中。当发生循环依赖的时候，Spring 可以从缓存中获取到早期暴露的 Bean 实例，从而打破循环。

Spring 创建 Bean 主要分为三个步骤：

1. 实例化，相当于 new 一个对象。
2. 属性注入，相当于使用对象的 setter 注入一些属性
3. 初始化。

对应的三级缓存则是：

1. **singletonObjects (一级缓存)**：存放完全实例化、初始化好的单例Bean。
2. **earlySingletonObjects (二级缓存)**：存放早期暴露的单例Bean，这些Bean已经被实例化，但还没有进行属性填充和初始化。
3. **singletonFactories (三级缓存)**：存放用于生成早期暴露的Bean的ObjectFactory对象。

如果出现了循环依赖，并且都是构造器注入的话，在第一步实例化时就会出现：实例化 A 时，需要注入对象 B，实例化 B 时，需要注入对象 A，但是 A 还没有实例化完成，所以出现了循环依赖，从而报错。

如果对象 B 是通过 setter 注入 A 的，则不会出现循环依赖。

所以加上缓存，Spring 创建 Bean 的步骤如下：

1. 实例化：Spring 使用反射创建 Bean A 的实例，此时 Bean A 只是一个「空壳」，它没有设置属性
2. 缓存早期暴露的 Bean：将刚刚创建好的 Bean A 存入**三级缓存**中 (以一个 ObjectFactory 的形式)。这个ObjectFactory的作用是，当其他 Bean (比如 Bean B) 需要依赖 Bean A 时，可以通过这个 ObjectFactory 获取到 Bean A 的早期实例。
3. 属性注入：Spring 为刚刚创建的 Bean A 的属性进行赋值。如果此时发现 Bean A 依赖于 Bean B，Spring 就会去创建 Bean B。
4. 创建 Bean B：在这个过程中，Spring 发现 Bean B 依赖于 Bean A，它首先会去**一级缓存**中寻找 Bean A，找不到则去二级缓存中寻找，最后去三级缓存中寻找，找不到则返回 null。
5. 从三级缓存中获取早期 Bean A：Spring 从三级缓存中获取 Bean A 的 ObjectFactory，并通过它获取到 Bean A 的早期实例。此时，Bean A 会被放入二级缓存中同时从三级缓存中移除，表示它**早期暴露**。
6. 继续创建 Bean B：Bean B 获取到了 Bean A 的实例，完成了实例化和属性注入。此时 Bean B 在创建完成后会被放入**一级缓存**中。
7. 继续创建 Bean A
8. Bean A 创建完成

### 三级缓存和二级缓存有什么区别

三级缓存和二级缓存都存放的是实例化但是没有进行属性填充的对象，那么为什么还需要它们两个一起呢？其实，三级缓存的存在是为了解决一个特殊情况：AOP (面向切面编程)。

如果 Spring 在实例化 Bean A 后，直接将其早期实例放入二级缓存，那么当 Bean B 依赖 Bean A 时，获取到的是没有经过 AOP 代理的原始 Bean A 实例。如果 Bean A 需要被 AOP 代理，那么最终放入一级缓存的将是代理后的 Bean A 实例，而 Bean B 依赖的却是原始的 Bean A 实例，这就可能导致问题。

三级缓存中的 ObjectFactory 提供了一个机会，可以在获取早期 Bean 实例时进行 AOP 代理。当 Bean B 通过 ObjectFactory 获取 Bean A 的早期实例时，ObjectFactory 会判断 Bean A 是否需要进行 AOP 代理。如果需要，就返回代理后的实例；如果不需要，就返回原始实例。这样就确保了 Bean B 获取到的 Bean A 实例是最终正确的实例（可能是原始实例，也可能是代理实例）。

二级缓存则不会检查 Bean 是否需要进行 AOP 代理，如果没有二级缓存，那么如果有多个对象依赖 Bean A，那么就需要多次在三级缓存中检查是否需要进行 AOP 代理。


## 2. Spring 模块

### Spring Core

Spring Core 是 Spring 框架的核心模块，提供了 IoC (控制反转) 和 DI (依赖注入) 的功能。它主要负责对象的创建、配置和生命周期管理。它是所有其他 Spring 模块的基础，别的模块都会依赖于它。

### Spring Beans

Spring Beans 负责管理 Bean 的定义和生命周期。通过 IoC 容器完成 Bean 的创建、依赖注入、初始化、销毁等操作。

> **Spring Core** 提供了 **IoC 容器** 的基础能力。它定义了 IoC 和 DI 的概念，以及如何实现这些概念的接口和核心类（比如 `BeanFactory` 和 `ApplicationContext`）。你可以把 Spring Core 看作是 IoC 容器的**骨架和核心机制**。
> 
> **Spring Beans** 模块则专注于 **Bean 的定义、管理和生命周期**。它提供了定义 Bean 的方式（例如 XML、注解、Java Config），以及在 IoC 容器中管理这些 Bean 的具体实现。它负责解析 Bean 的配置信息，创建 Bean 的实例，执行依赖注入，以及管理 Bean 的初始化和销毁过程。你可以把 Spring Beans 看作是 IoC 容器中**具体处理和管理对象（Bean）的部分**。

### Spring Context

Spring Context 是基于 Spring Core 和 Spring Bean 的高级容器，代表 Spring 容器运行时的环境，包含了 Spring 容器管理的所有 Bean，以及容器提供的各种功能和服务。

`ApplicationContext` 是 Spring Context 的一种实现。

### Spring Expression Language (SpEL)

SeEL 是 Spring 提供的一种表达式语言，用于在运行时查询和操作对象图。它可以在 Spring 配置文件中使用，也可以在 Java 代码中使用。

```xml
<bean id="myBean" class="com.example.MyBean"> 
	<property name="someValue" value="#{anotherBean.someProperty}"/>
</bean>
```

在上面的例子中，`#{anotherBean.someProperty}` 是一个 SpEL 表达式，它会在运行时被解析为 `anotherBean` 的 `someProperty` 属性的值。在 Spring Boot 中，上面的配置通常会被注解 `@Value("#{anotherBean.someProperty}")` 替代。

### Spring AOP

提供面向切面编程的功能，可以在方法执行**前后**或**抛出异常**时动态插入额外的逻辑，比如日志记录、权限验证、事务管理等。

### Spring JDBC/ORM/Transaction

- Spring JDBC: 简化了原生 JDBC 的操作，提供模板方法来管理连接、资源的释放和异常处理。
- Spring ORM: 支持与主流 ORM 框架 (如 Hibernate、JPA、MyBatis 等)集成，简化持久层的开发。
- Spring Transaction: 提供声明式 (@Transactional 注解) 和编程式 (注入 TransactionTemplate) 的事务管理机制，与数据库操作密切结合。

### Spring Web/MVC

- Spring Web: 提供基础的 Web 开发支持，包括 Servlet API 的集成，适用于构建 MVC 架构。
- Spring MVC: 实现了 MVC 模式的框架，用于构建基于 HTTP 请求的 Web 应用，支持注解驱动的 Web 开发。

## 3. Spring IoC

Spring IoC (控制反转) 是 Spring 框架的核心概念之一。它是通过 DI (依赖注入) 实现的。

- 控制反转表示将对象的创建和依赖关系交由 Spring 容器管理，而不是由程序代码直接控制。这种机制使得程序更加灵活和解耦，提升了代码的可维护性和灵活性。
- 依赖注入是实现控制反转的一种方式，通过构造器注入、Setter 注入或接口注入，将对象所需要的依赖传给它。

控制反转，其中控制表示**控制对象的创建**。反转表示**反转「创建对象并注入依赖」** 这个动作。

## 3. BeanFactory

BeanFactory 其实就是 IoC 的底层容器，它是 Spring Ioc 的最基础的实现，提供了最基本的 IoC 功能。它负责创建和管理 Bean 的生命周期。

- BeanFactory 负责从配置源中读取 Bean 的定义，并创建、管理这些 Bean 的生命周期
- BeanFactory 是延迟初始化的，只有 Bean 在用到的时候才会实例化这个 Bean

## 4. @Bean 和 @Component 的区别

- @Bean 注解通常用于配置类 (@Configuration 注解) 中的方法上，表示这个方法会返回一个 Bean 实例。@Bean 注解是 Java 配置的方式，通常用于第三方库的集成。
- @Component 注解通常用于类上，表示这个类是一个 Spring 管理的组件。@Component 注解是基于类的扫描方式，通常用于自定义的 Bean。

```java
@Configuration
public class AppConfig {
	@Bean
	public DataSource dataSource() {
		DriverManagerDataSource dataSource = new DriverManagerDataSource();
		// 一些配置操作
		return dataSource;
	}
}

@Component
public class Person {
	// 一些属性
}
```

## 5. @Primary 注解

当 Spring 容器中存在多个可以被注入的 Bean 时，使用 @Primary 注解可以指定优先注入哪个 Bean。

```java
@Component
@Primary
public class ServiceImpl1 implements IService {

}

@Component
public class ServiceImpl2 implements IService {

}

@Component
public class MyController {
	@Autowired
	private IService service; // 默认注入 ServiceImpl1
}
```

## 6. @Value 注解

在 Spring 框架中，@Value 用于注入外部的配置值到 Spring 管理的 Bean 中。通过 @Value 注解，可以将属性文件、环境变量、系统属性等外部资源中的值注入到 Spring Bean 的字段、方法中。

```properties
app.name=MyApp
app.version=1.0.0
```

```java
@Component
public class AppConfig {
	@Value("${app.name}")
	private String appName;

	@Value("${app.version}")
	private String appVersion;

	public String getAppName() {
		return appName;
	}

	public String getAppVersion() {
		return appVersion;
	})
}
```

- @Value("Jack") 注入常量
- @Value("classpath:com/test/config.xml") 注入资源文件 (Resource 类型)
- @Value("${app.name}") 注入配置文件中的属性值
- @Value("#{user.name}") 注入对象属性

## 7. @Profile 注解

在开发中，通常会有不同的环境，最基本的，可以分为开发环境 (dev) 和生产环境 (prod)。针对于这些不同的环境，也需要不同的配置方案。@

`@Profile({profile})` 注解用于指定这个配置类在哪个环境下生效 (profile = dev、prod)。

那么如何指定当前的环境呢？通常会在 `application` 配置文件中，通过配置项 `spring.profiles.active` 来指定，在制定完成之后，Spring 会尝试加载 `application-{profile}` 配置文件。

所以通常 `application` 中存放基础的配置，`application-{profile}` 中存放针对不同环境的配置。至于配置类则用 `@Profile` 进行区分。

## 8. @PostConstruct 和 @PreDestroy 注解

@PostConstruct 注解用于标记方法，当 Spring 容器完成 Bean 注入之后，就会调用 @PostConstruct 标记的方法，主要用于参数的初始化。

@PreDestroy 注解用于标记方法，Spring 容器在关闭时，会自动调用 @PreDestroy 标记的方法，主要用于资源的释放。

> 在使用 Spring Boot 开发时，Spring Boot 框架本身极大地简化了流程，所以很多时候，我们不需要手动使用 @PreDestroy 和 @PostConstruct 注解来进行资源的释放和初始化，因为 Spring Boot 会自动处理这些事情。

## 9. @RequestBody 和 @ReponseBody 注解

- @RequestBody 注解把 HTTP 请求体中的数据转换为 Java 对象，绑定到方法的参数上。请求体中的类型可以是 Json、XML 或者自定义数据等，Spring Boot 会根据请求中的 Content-Type 来使用对应的转换器进行转换
- @ResponseBody 注解把 Java 对象作为返回结果写入到 HTTP 响应体中。通常用于返回 Json、XML 或者自定义数据等，Spring Boot 会根据响应中的 Content-Type 来使用对应的转换器进行转换

> 在定义转换器时，需要指明这个转换器支持的 Content-Type，例如 Json 转换器需要指明它支持 "application/json"。然后当请求或者响应的 Content-Type 符合类型时，Spring Boot 就会自动使用这个转换器进行转换。

## 10. @PathVariable 和 @RequestParam 注解

- @PathVariable 注解用于将请求 URL 路径中捕获变量，并将其绑定到方法参数上。

例如，对于请求路径 "http://example.com/user/123" 为了获取到路径中的 "123" 部分，可以使用 @PathVariable 注解：

```java
@GetMapping("/user/{id}")
public Result<User> getUserById(@PathVariable("id", required = true) String userId) {

}
```

- @RequestParam 注解用于从请求的查询参数中提取数据，并将它们绑定到方法参数上。

例如，对于请求路径 "http://example.com/user?id=123" 为了获取到查询参数中的 "id" 部分，可以使用 @RequestParam 注解：

```java
@GetMapping("/user")
public Result<User> getUserById(@RequestParam("id") String userId) {

}
```

## 11. @ModelAttribute 注解

@ModelAttribute 有两个用法：

1. 用于方法参数上，表示将请求的参数或者**表单数据**绑定到控制器处理方法的参数上。它的功能与 @RequestBody 类似，不过 @ModelAttribute 更加专攻表单数据的绑定。
2. 用在方法上，表示在控制器方法执行之前，先执行这个方法。这个方法通常用于初始化一些数据，或者设置一些公共的属性。

```java
@Controller
public class UserController {

    @ModelAttribute("roles")
    public List<String> populateRoles() {
        // 模拟从数据库中获取角色数据
        return Arrays.asList("Admin", "User", "Guest");
    }

    @GetMapping("/showForm")
    public String showForm(Model model) {
        model.addAttribute("user", new User());
        return "userForm";
    }
}
```

上面的 `showForm` 在执行前，会执行 `populateRoles` 方法，获得一个 `Model`，它的 `roles` 属性的值是 `populateRoles` 方法的返回值。这个 `Model` 会传递给视图进行渲染。这个 `Model` 的生命周期是和当前请求的生命周期一致的。

## 12. @ExceptionHandler 注解

@ExceptionHandler 注解是 Spring MVC 提供的用于处理 Controller 中抛出的异常的注解。它可以允许开发者定义一个专门用于处理特定异常的方法，示例如下：

```java
@ExceptionHandler({UserNotFoundException.class, InvalidUserException.class})
public String handlerUserException(Exception ex, Model model) {
	
}
```

> 在 Spring Boot 中，推荐的异常处理方式也是使用 @ExceptionHandler 注解，它主要有两种用法，一是直接在 Controller 中使用，二是配合 @ControllerAdvice 注解进行全局异常处理。

```java
@RestController
public class MyController {

    @GetMapping("/test")
    public String testException() {
        throw new RuntimeException("这是一个运行时异常");
    }

	// 直接在 Controller 中使用 @ExceptionHandler
    @ExceptionHandler(RuntimeException.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR) // 设置响应状态码
    public String handleRuntimeException(RuntimeException ex) {
        return "捕获到运行时异常: " + ex.getMessage();
    }
}
```

```java
@ControllerAdvice
public class GlobalExceptionHandler {
	// 全局异常处理
    @ExceptionHandler(RuntimeException.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public String handleRuntimeException(RuntimeException ex) {
        return "全局捕获到运行时异常: " + ex.getMessage();
    }

    @ExceptionHandler(IllegalArgumentException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST) // 设置响应状态码
    public String handleIllegalArgumentException(IllegalArgumentException ex) {
        return "全局捕获到非法参数异常: " + ex.getMessage();
    }
}

@RestController
public class AnotherController {

    @GetMapping("/another")
    public String anotherMethod(@RequestParam int id) {
        if (id <= 0) {
            throw new IllegalArgumentException("ID 必须大于 0");
        }
        return "ID: " + id;
    }
}

```

## 13. @ResponseStatus 注解

@ResponseStatus 注解用于定义一个方法或者异常类所对应的 HTTP 状态码和原因。在使用这个注解时，Spring 会自动把这个状态码和原因添加到 HTTP 响应中。

```java
@ResponseStatus(value = HttpStatus.NOT_FOUND, reason = "资源未找到")
public class UserNotFoundException extends RuntimeException {
    // 异常类实现
}
```

1. 当 @ResponseStatus 用在异常类上时，Spring 会在抛出这个异常时自动返回指定的状态码和原因。
2. 当 @ResponseStatus 用在方法上时，Spring 会在调用这个方法时返回指定的状态码和原因，所以这个注解应该用在 Controller 或者 ExceptionHandler 这种能够直接与前端进行交互的方法上。

## 14. @RequestHeader 和 @CookieValue 注解

- @RequestHeader 注解用于提取 HTTP 请求头中的值，并将其注入到 Controller 方法的参数中。

```java
@GetMapping("/header-info")
public String getHeaderInfo(@RequestHeader("User-Agent") String userAgent) {
    // 使用 userAgent 进行业务处理
    return "headerInfoView";
}

```

- @CookieValue 注解用于从 HTTP 请求的 Cookie 中提取值，并将其注入到 Controller 方法的参数中。

```java
@GetMapping("/cookie-info")
public String getCookieInfo(@CookieValue("sessionId") String sessionId) {
    // 使用 sessionId 进行业务处理
    return "cookieInfoView";
}
```

> Cookie 也是存放在请求头中的，@RequestHeader("Cookie") 也是可以直接获取到整个 Cookie 的，例如 Cookie: "sessionId=123456; userId=321"。不过如果使用 Cookie("sessionId") 则可以直接进行 Cookie 的解析，得到 "123456"。

--- 

## Ex 1. @Transaction 注解

`@Transactional` 注解的原理主要是基于 AOP 和动态代理技术。

当给一个方法加上 `@Transactional` 注解后，框架不会直接执行方法的代码，而是会创建一个代理对象。在调用这个方法时，其实是在调用代理对象的这个方法，代理会在方法前后进行事务处理。

1. 方法调用前：检查事务的传播行为，以及当前是否有活动的事务，然后决定是否开启事务
2. 方法执行
3. 方法调用后：根据是否抛出异常来决定是执行提交事务还是回滚事务

### @Transactional 失效场景

#### 自调用

```java
@Service
public class MyService {

    public void methodA() {
        this.methodB(); // <--- 这里 @Transactional 会失效
    }

    @Transactional
    public void methodB() {
        // 数据库操作
    }
}
```

在上面的例子中，`methodA()` 调用的不是代理对象，此时事务是不会生效的；正确的做法应该是先注入 MyService，然后调用注入的对象的 `methodB`。

#### 访问修饰符问题

`@Transactional` 注解只对 `public` 生效。对于 JDK 代理，由于需要实现接口，所以非 `public` 方法是不会被代理的；对于 `CGLIB` 代理，`protected` 理论上可以被代理，但是并不推荐。

#### 主动处理了异常

`@Transactional` 会在捕获到异常或者错误时进行事务回滚，而如果在 `@Transactional` 方法中手动处理了异常，那么事务就不会回滚。

#### 在事务中启用了线程

在默认情况下，线程中出现了异常是不会影响到主线程的，所以对于外部的 `@Transactional` 来说，事务也不会回滚。

#### 事务传播行为配置错误

`@Transactional` 注解的传播行为可以配置为不同的值，例如 `REQUIRED`、`REQUIRES_NEW` 等。如果配置错误，可能会导致事务没有按照预期进行。

## Ex 2. Spring Bean 生命周期

Spring 中 Bean 的生命周期主要可以分为 5 个阶段：

1. 实例化：Spring 利用反射创建 Bean 实例，此时 Bean 中的属性都还没有被赋值
2. 属性赋值：通过 @Value 注解、@Autowired 注解等方式，将 Bean 的属性进行赋值
3. 初始化：在 xml 中通过 `init-method` 指定初始化方法，在 Spring Boot 中，通过 `@PostConstruct` 注解指定初始化方法
4. 使用 Bean
5. 销毁：在 xml 中通过 `destory-method` 指定销毁方法，在 Spring Boot 中，通过 `@PreDestroy` 注解指定销毁方法

更详细地，在第三步初始化操作前后，可以多处两步：

- BeanPostProcessor 的 `postProcessBeforeInitialization` 方法
- 初始化
- BeanPostProcessor 的 `postProcessAfterInitialization` 方法
