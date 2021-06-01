----------------------------
Spring-Boot 基本配置		|
----------------------------
	# 入口类配置
		* 一般应用都会有一个 XxxxApplication 类,该就一个 main 方法,并且该类被 @SpringBootApplication 标注
		* @SpringBootApplication 这是一个组合注解,组合了以下的注解
			* @Configuration 
			* @EnableAutoConfiguration 
			* @ComponentScan 

		* spring boot 还会自动扫描该注解标识类所在包,同级包,以及子包中Bean
			* 如果是JPA项目,还可以扫描标注@Entity的实体类

		* 建议入口类放置的位置在 grouId + arctifactId 组合的包名下
	
	# 关闭特定的自动配置
		* @SpringBootApplication 的一个属性
			Class<?>[] exclude() default {};

		* 该属性中的 Class 配置,会被关闭自动配置
			@SpringBootApplication(exclude = {DataSourceConfiguration.class});
	
	# 定制Banner
		* spring 启动的时候会有一个默认的启动图案
			  .   ____          _            __ _ _
			 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
			( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
			 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
			  '  |____| .__|_| |_|_| |_\__, | / / / /
			 =========|_|==============|___/=/_/_/_/
			 :: Spring Boot ::        (v1.5.1.RELEASE)
		* 在 src/main/resources 下新建文件:banner.txt
		* 打开网站,创作出喜欢的样式:http://patorjk.com/software/taag/#p=display&f=Graffiti&t=Type%20Something%20 
		* 复制到 banner.txt中,重启程序
		* 关闭banner,有两张方法
			1,在main里面进行修改
				SpringApplication springApplication = new SpringApplication(SampleController.class);
				//设置Banner为关闭模式
				springApplication.setBannerMode(Banner.Mode.OFF);
				springApplication.run(args);

			2,使用 fluent API 进行修改
				new SpringApplicationBuilder(SampleController.class).bannerMode(Banner.Mode.OFF).run(args);
		* 而且,Banner里面是可以使用一些特殊的变量 ${name}
				
				
	# 配置文件
		* spring boot 使用一个全局的配置文件:application.properties / application.yml
		* 该文件应该是在 src/main/resources 目录,或者是classpath:/config 目录下
		* 可选配置项(很多)
			server.port				//端口
			server.context-path		//上下文路径


	# 使用 xml 配置
		* spring boot 提倡无xml配置,但是在实际项目中,可能有特殊的需求
		* 使用 @ImportResource 来加载xml配置,该注解是 spring 的注解
		* @ImportResource({"classpath:xxx.xml","classpath:yyy.xml"});
				
	
	# 随机值的生成
		* 有时候我们项目需要生成随机的一些值,我们并不用手动的去写代码
		* RandomValuePropertySource 可以用来生成测试所需要的各种不同类型的随机值，从而免去了在代码中生成的麻烦。
		* RandomValuePropertySource 可以生成数字和字符串。数字的类型包含 int 和 long，可以限定数字的大小范围。
		* 在配置文件中以"random."作为前缀的配置属性
			user.id=${random.value}
			user.count=${random.int}
			user.max=${random.long}
			user.number=${random.int(100)}
			user.range=${random.int[100, 1000]}

----------------------------
Spring-Boot 外部配置		|
----------------------------
	# spring boot 允许开发人员使用属性文件、YAML 文件、环境变量和命令行参数来定义优先级不同的配置值。
	# Spring Boot 提供的 SpringApplication 类会搜索并加载 application.properties 文件来获取配置属性值。SpringApplication 类会在下面位置搜索该文件。
		1,当前目录的"/config"子目录。
		2,当前目录。
		3,classpath 中的"/config"包。
		4,classpath
		* 上面的顺序也表示了该位置上包含的属性文件的优先级(从高到低)

	# 配置优先级顺序比较复杂。按照优先级从高到低的顺序
		1,命令行参数。
		2,通过 System.getProperties() 获取的 Java 系统参数。
		3,操作系统环境变量。
		4,从 java:comp/env 得到的 JNDI 属性。
		5,通过 RandomValuePropertySource 生成的“random.*”属性。
		6,应用 Jar 文件之外的属性文件。
		7,应用 Jar 文件内部的属性文件。
		8,在应用配置 Java 类（包含"@Configuration"注解的 Java 类）中通过"@PropertySource注解声明的属性"文件。
		9,通过 "SpringApplication.setDefaultPrope"ties"声明的默认属性。

	# 命名行参数配置
		* spring boot 可以基于jar包运行,打包成jar后可以在运行命令后添加参数
			java -jar xxx.jar --server.port=9090
			java -jar xxx.jar --spring.profiles.active=remote
		* SpringApplication 类默认会把以"--"开头的命令行参数转化成应用中可以使用的配置参数，如 "--name=Alex" 会设置配置参数 "name" 的值为 "Alex"。
		* 如果不需要这个功能，可以通过  SpringApplication.setAddCommandLineProperties(false) 禁用解析命令行参数。

	# 常规属性配置
		* 在应用程序中,直接使用 @Value("${key}"),就可以读取到 application.properties 中的数据
	
	# 类型安全的配置
		* 使用 @ConfigurationProperties 和一个bean及其属性进行关联
		* Demo
			1,在 application.properties 中添加属性
				user.name=kevin

			2,使用注解,默认的使用 application.properties
				@ConfigurationProperties(prefix="user")
				class User{
					private String name;
				}

	# 日志配置
		* spring boot 支持 java.util.Logging,Log4j,lo4j2和logback作为日志框架
		* 默认,使用logback作为日志框架

		* 默认的配置文件配置(不建议使用,不够灵活,对log4j2不够友好)
			> 配置日志文件地址
				logging.file=d:/mylog/log.log
			> 配置日志输出级别
				logging.level.org.springframework.web=DEBUG		
				logging.level.*=INFO

		* 引用外部文件的配置(推荐使用)
			> spring boot 会默认加载:classpath:logback-spring.xml或者classpath:logback-spring.groovy

			> 使用自定义配置文件,指定配置文件的地址
				logging.config=classpath:logback-xxx.xml
				* 注意：不要使用logback这个来命名，否则spring boot将不能完全实例化
				* 配置文件有两种方式创建

			> 1,使用基于spring boot的配置
				<?xml version="1.0" encoding="UTF-8"?>
					<configuration>
					<include resource="org/springframework/boot/logging/logback/base.xml"/>
					<logger name="org.springframework.web" level="DEBUG"/>
				</configuration>

			> 2,自定义的配置
				* 略,自己去看logback的配置
								
			
	# profile 环境配置
		* 不同的环境,使用不同的profile
		* 每个profile的格式固定
			application-{name}.properties

		* application.properties文件中指定要加载的文件
			spring.profiles.active={name}
	
