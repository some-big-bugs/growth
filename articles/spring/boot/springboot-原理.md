--------------------------
Springboot-原理				|
--------------------------


--------------------------
Springboot-自动配置原理		|
--------------------------
	# 关于spring-boot自动配置的源码都在:spring-boot-autoconfigure-1.5.1.RELEASE.jar 中
	# 如果想看到spring-boot自动为我们做了哪些配置,可以在启动的时候添加参数,也可以在全局配置文件中添加参数
		java -jar xxx.jar --debug
		debug=true
		* Positive matches: 以下为启用的自动配置
		* Negative matches:	以下为未启用的自动配置
		
	0,@SpringBootApplication 注解
	1,@EnableAutoConfiguration 注解
	2,@Import(EnableAutoConfigurationImportSelector.class) 注解
	3,EnableAutoConfigurationImportSelector 类
		# 使用 SpringFactoriesLoader.loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader);方法来扫描具有 META-INF/spring.factories 的jar包
			* public static List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader)
			* spring-boot-autoconfigure-1.5.1.RELEASE.jar 中就有 META-INF/spring.factories
		# META-INF/spring.factories 里面声明有一些自动配置项
	
	4,随便打开一个 XxxxAutoConfiguration 类,一般都会有以下注解,在 org.springframework.boot.autoconfigure.condition 包下
		@ConditionalOnBean
			* 当容器有指定的bean的条件下
		@ConditionalOnClass
			* 当容器路径下有指定类的条件下
		@ConditionalOnCloudPlatform
		@ConditionalOnExpression
			* 基于SpEL表达式作为判断条件
		@ConditionalOnJava
			* 基于JVM版本作为判断条件
		@ConditionalOnJndi
			* 在JNDI存在的条件下查找指定的位置
		@ConditionalOnMissingBean
			* 当容器里面没有指定Bean的情况下
		@ConditionalOnMissingClass
			* 当类路径下没有指定的类条件下
		@ConditionalOnNotWebApplication
			* 当前项目不是Web项目的条件下
		@ConditionalOnProperty
			* 指定的属性是否有指定的值
		@ConditionalOnResource
			* 类路径是否有指定的值
		@ConditionalOnSingleCandidate
			* 当指定的 Bean 在容器中只有一个,或者虽然有多个,但是指定首选的Bean
		@ConditionalOnWebApplication
			* 当前项目是WEB项目的条件下

		* 这堆注解都是组合了 @Conditional(spring) 元注解,只是使用了不同的条件
	
--------------------------
Springboot-自定义AutoConfiguration |
--------------------------

	1,准备配置Bean,并且读取properties文件
		* 一般都是以: xxxProperties 命名
		* 如果是单独抽离出来模块(maven),仅仅需要依赖spring boot的 autoconfigure 模块:
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-autoconfigure</artifactId>

	2,创建 XxxxAutoConfiguration 类,会通过该类的一些注解来完成是否要注册一些组件
		* 标识该类是一个配置类 @Configuration
		* 把 创建的 xxxProperties 加入到IOC EnableConfigurationProperties(xxxProperties.class) ,就可以在页面中 @Autowired 注入使用
		* 某个注册 @Bean 的方法上去使用注解进行判断(上面的那一堆了就是)
		* 把返回的Bean注入到IOC
			
		
	3,在 src/main/resource 目录下创建文件夹和文件
		META-INF/spring.factories


		org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
		com...xxxx,\
		com...x...,\