> 这篇文章由多个资料和文档整理而来

# Spring 系列注解

[TOC]

# Spring

## Bean 声明和注入

**声明bean的注解**

- @Component 组件，没有明确的角色
- @Service 在业务逻辑层使用（service层）
- @Repository 在数据访问层使用（dao层）
- @Controller 在展现层使用，控制器的声明（C）

**注入bean的注解**

- @Autowired：由Spring提供
- @Inject：由JSR-330提供
- @Resource：由JSR-250提供

都可以注解在set方法和属性上，推荐注解在属性上（一目了然，少写代码）。

## 配置相关

- @Configuration 声明当前类为配置类，相当于xml形式的Spring配置（类上）
- @Bean 注解在方法上，声明当前方法的返回值为一个bean，替代xml中的方式（方法上）
- @Configuration 声明当前类为配置类，其中内部组合了@Component注解，表明这个类是一个bean（类上）
- @ComponentScan 用于对Component进行扫描，相当于xml中的（类上）
- @WishlyConfiguration 为@Configuration与@ComponentScan的组合注解，可以替代这两个注解

## 切面（AOP）


使用@After、@Before、@Around定义建言（advice），可直接将拦截规则（切点）作为参数。


- @Aspect 声明一个切面（类上）
- @After 在方法执行之后执行（方法上）
- @Before 在方法执行之前执行（方法上）
- @Around 在方法执行之前与之后执行（方法上）
- @PointCut 声明切点

在java配置类中使用@EnableAspectJAutoProxy注解开启Spring对AspectJ代理的支持（类上）

## @Bean的属性支持

- @Scope 设置Spring容器如何新建Bean实例（方法上，得有@Bean）其设置类型包括：
  - Singleton （单例,一个Spring容器中只有一个bean实例，默认模式）,
  - Protetype （每次调用新建一个bean）,
  - Request （web项目中，给每个http request新建一个bean）,
  - Session （web项目中，给每个http session新建一个bean）,
  - GlobalSession（给每一个 global http session新建一个Bean实例）

- @StepScope 在Spring Batch中还有涉及
- @PostConstruct 由JSR-250提供，在构造函数执行完之后执行，等价于xml配置文件中bean的initMethod
- @PreDestory 由JSR-250提供，在Bean销毁之前执行，等价于xml配置文件中bean的destroyMethod

6、@Value注解

@Value 为属性注入值（属性上）

支持如下方式的注入：

- 注入普通字符
- 注入操作系统属性
- 注入表达式结果
- 注入其它bean属性
- 注入文件资源
- 注入网站资源
- 注入配置文件

## 环境切换

- @Profile 通过设定Environment的ActiveProfiles来设定当前context需要使用的配置环境。（类或方法上）
- @Conditional Spring4中可以使用此注解定义条件话的bean，通过实现Condition接口，并重写matches方法，从而决定该bean是否被实例化。（方法上）

## 异步相关

- @EnableAsync 配置类中，通过此注解开启对异步任务的支持，叙事性AsyncConfigurer接口（类上）
- @Async 在实际执行的bean方法使用该注解来申明其是一个异步任务（方法上或类上所有的方法都将异步，需要@EnableAsync开启异步任务）

## 定时任务相关

- @EnableScheduling 在配置类上使用，开启计划任务的支持（类上）
- @Scheduled 来申明这是一个任务，包括cron,fixDelay,fixRate等类型（方法上，需先开启计划任务的支持）

## @Enable*注解说明

这些注解主要用来开启对xxx的支持。

- @EnableAspectJAutoProxy 开启对AspectJ自动代理的支持
- @EnableAsync 开启异步方法的支持
- @EnableScheduling 开启计划任务的支持
- @EnableWebMvc 开启Web MVC的配置支持
- @EnableConfigurationProperties 开启对@ConfigurationProperties注解配置Bean的支持
- @EnableJpaRepositories 开启对SpringData JPA Repository的支持
- @EnableTransactionManagement 开启注解式事务的支持
- @EnableTransactionManagement 开启注解式事务的支持
- @EnableCaching 开启注解式的缓存支持

## 测试相关注解

- @RunWith 运行器，Spring中通常用于对JUnit的支持
- @ContextConfiguration 用来加载配置ApplicationContext，其中classes属性用来加载配置类

# Spring MVC

- @EnableWebMvc 在配置类中开启Web MVC的配置支持，如一些ViewResolver或者MessageConverter等，若无此句，重写WebMvcConfigurerAdapter方法（用于对SpringMVC的配置）。
- @Controller 声明该类为SpringMVC中的Controller
- @RequestMapping 用于映射Web请求，包括访问路径和参数（类或方法上）
- @ResponseBody 支持将返回值放在response内，而不是一个页面，通常用户返回json数据（返回值旁或方法上）
- @RequestBody 允许request的参数在request体中，而不是在直接连接在地址后面。（放在参数前）
- @PathVariable 用于接收路径参数，比如@RequestMapping(“/hello/{name}”)申明的路径，将注解放在参数中前，即可获取该值，通常作为Restful的接口实现方法。
- @RestController 该注解为一个组合注解，相当于@Controller和@ResponseBody的组合，注解在类上，意味着，该Controller的所有方法都默认加上了@ResponseBody。
- @ControllerAdvice 通过该注解，我们可以将对于控制器的全局配置放置在同一个位置，注解了@Controller的类的方法可使用@ExceptionHandler、@InitBinder、@ModelAttribute注解到方法上，
- @ExceptionHandler 用于全局处理控制器里的异常
- @InitBinder 用来设置WebDataBinder，WebDataBinder用来自动绑定前台请求参数到Model中。
- @ModelAttribute 本来的作用是绑定键值对到Model里，在@ControllerAdvice中是让全局的- @RequestMapping都能获得在此处设置的键值对。



## 注解及介绍

### 1、@SpringBootApplication

这是 Spring Boot 最最最核心的注解，用在 Spring Boot 主类上，标识这是一个 Spring Boot 应用，用来开启 Spring Boot 的各项能力。

其实这个注解就是 `@SpringBootConfiguration、@EnableAutoConfiguration、@ComponentScan` 这三个注解的组合，也可以用这三个注解来代替 @SpringBootApplication 注解。@SpringBootApplication 默认扫描和本类在一个层级下的所有包及其子包

### 2、@EnableAutoConfiguration

允许 Spring Boot 自动配置注解，开启这个注解之后，Spring Boot 就能根据当前类路径下的包或者类来配置 Spring Bean。

如：当前类路径下有 Mybatis 这个 JAR 包，MybatisAutoConfiguration 注解就能根据相关参数来配置 Mybatis 的各个 Spring Bean。

### 3、@Configuration

这是 Spring 3.0 添加的一个注解，用来代替 applicationContext.xml 配置文件，所有这个配置文件里面能做到的事情都可以通过这个注解所在类来进行注册。替代了xml形式的配置文件，用配置类开发

### 4、@SpringBootConfiguration

这个注解就是 @Configuration 注解的变体，只是用来修饰是 Spring Boot 配置而已，或者可利于 Spring Boot 后续的扩展。

### 5、@ComponentScan

这是 Spring 3.1 添加的一个注解，用来代替配置文件中的 component-scan 配置，开启组件扫描，即自动扫描包路径下的 @Component 注解进行注册 bean 实例到 context 中。

### 6、@Conditional

这是 Spring 4.0 添加的新注解，用来标识一个 Spring Bean 或者 Configuration 配置文件，当满足指定的条件才开启配置。

### 7、@ConditionalOnBean

组合 @Conditional 注解，当容器中有指定的 Bean 才开启配置。

### 8、@ConditionalOnMissingBean

组合 @Conditional 注解，和 @ConditionalOnBean 注解相反，当容器中没有指定的 Bean 才开启配置。

### 9、@ConditionalOnClass

组合 @Conditional 注解，当容器中有指定的 Class 才开启配置。

### 10、@ConditionalOnMissingClass

组合 @Conditional 注解，和 @ConditionalOnMissingClass 注解相反，当容器中没有指定的 Class 才开启配置。

### 11、@ConditionalOnWebApplication

组合 @Conditional 注解，当前项目类型是 WEB 项目才开启配置。

当前项目有以下 3 种类型。

```
enum Type {
/**
 * 匹配全部web application
 */
ANY,

/**
 *只匹配servlet web applicaiton
 */
SERVLET,

/**
 * 只匹配reactice-based web application
 */
REACTIVE
}
```

### 12、@ConditionalOnNotWebApplication

组合 @Conditional 注解，和 @ ConditionalOnWebApplication 注解相反，当前项目类型不是 WEB 项目才开启配置。

### 13、@ConditionalOnProperty

组合 @Conditional 注解，当指定的属性有指定的值时才开启配置。

### 14、@ConditionalOnExpression

组合 @Conditional 注解，当 SpEL 表达式为 true 时才开启配置。

### 15、@ConditionalOnJava

组合 @Conditional 注解，当运行的 Java JVM 在指定的版本范围时才开启配置。

### 16、@ConditionalOnResource

组合 @Conditional 注解，当类路径下有指定的资源才开启配置。

### 17、@ConditionalOnJndi

组合 @Conditional 注解，当指定的 JNDI 存在时才开启配置。JDNI（Java 命名与目录接口 Java Naming and Directory Interface）

### 18、@ConditionalOnCloudPlatform

组合 @Conditional 注解，当指定的云平台激活时才开启配置。

### 19、@ConditionalOnSingleCandidate

组合 @Conditional 注解，当指定的 class 在容器中只有一个 Bean，或者同时有多个但为首选时才开启配置。

### 20、@ConfigurationProperties

用来加载额外的配置（如 .properties 文件），可用在 @Configuration 注解类，或者 @Bean 注解方法上面。

### 21、@EnableConfigurationProperties

一般要配合 @ConfigurationProperties 注解使用，用来开启对 @ConfigurationProperties 注解配置 Bean 的支持。

### 22、@AutoConfigureAfter

用在自动配置类上面，表示该自动配置类需要在另外指定的自动配置类配置完之后。

如 Mybatis 的自动配置类，需要在数据源自动配置类之后。

```
@AutoConfigureAfter(DataSourceAutoConfiguration.class)
public class MybatisAutoConfiguration {
}
```

### 23、@AutoConfigureBefore

这个和 @AutoConfigureAfter 注解使用相反，表示该自动配置类需要在另外指定的自动配置类配置之前。

### 24、@Import

这是 Spring 3.0 添加的新注解，用来导入一个或者多个 @Configuration 注解修饰的类，这在 Spring Boot 里面应用很多。

### 25、@ImportResource

这是 Spring 3.0 添加的新注解，用来导入一个或者多个 Spring 配置文件，这对 Spring Boot 兼容老项目非常有用，因为有些配置无法通过 Java Config 的形式来配置就只能用这个注解来导入。

## 读取配置方式汇总

### 1.读取application文件

在application.yml或者properties文件中添加:

```
user.name = 狼王
user.age = 25
user.sex = man 
@value 方式
@Component
public class User {

    @Value("${user.name}")
    private String name;
    @Value("${user.age}")
    private int age;
    @Value("${user.sex}")
    private boolean sex;

    public User() {
    }

    public User(String name, int age, boolean man) {
        this.name = name;
        this.age = age;
        this.man = man;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public boolean isMan() {
        return man;
    }

    public void setMan(boolean man) {
        this.man = man;
    }
}
```

### 2.@ConfigurationProperties注解读取方式

```
@Component
@ConfigurationProperties(prefix = "user")
public class User2 {
    private String name;

    private int age;

    private boolean sex;

    public User2() {
    }

    public User2(String name, int age, boolean man)     {
        this.name = name;
        this.age = age;
        this.man = man;
    }
}
```

## 3.读取指定文件

资源目录下建立config/myConfig.properties:

```
db.username=root
db.password=123
```

### @PropertySource+@Value注解读取方式

```
@Component
@PropertySource(value = {"config/myConfig.properties"})
public class User2 {

    @Value("${db.userName}")
    private String userName;
    @Value("${db.userName}")
    private String passWord;

    public User2() {
    }

    public User2(String userName,String passWord){
        this.userName = userName;
        this.passWord = passWord;
    }
}
```

也可以通过`@PropertySource+@ConfigurationProperties`注解读取方式