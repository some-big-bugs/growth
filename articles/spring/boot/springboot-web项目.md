--------------------------------
Spring-boot WEB项目				|
--------------------------------
	# Spring boot 对WEB的开发提供的支持
	# spring-boot-starter-web,提供了嵌入式的Tomcat和Springmvc的依赖
	# Web相关的自动配置在:spring-boot-autoconfigure.jar 的 org.springframework.boot.autoconfigure.web 下
		ServerPropertiesAutoConfiguration 和 ServerProperties
			* 自动配置内嵌Servlet容器
		
		HttpEncodingAutoConfiguration 和 HttpEncodingProperties
			* 自动配置http的编码
		
		MultipartAutoConfiguration 和 MultipartProperties
			* 自动配置上传文件的属性

		JacksonHttpMessageConvertersConfiguration 
			* 配置,mappingJackson2HttpMessageConverter 和 mappingJackson2XmlHttpMessageConverter
		
		WebMvcAutoConfiguration 和 WebMvcProperties
			* 配置spring mvc


--------------------------------
Spring-boot 拦截器				|
--------------------------------
	# spring boot拦截器默认有 
		HandlerInterceptorAdapter
		AbstractHandlerMapping
		UserRoleAuthorizationInterceptor
		LocaleChangeInterceptor
		ThemeChangeInterceptor


	2.配置spring mvc的拦截器 WebMvcConfigurerAdapter 
		* 自定义类继承 WebMvcConfigurerAdapter 
			* 该类作为配置类,应该添加 :@Configuration 注解
			* 也可以直接让 @SpringBootApplication 去实现
		
		* 覆写 addInterceptors 方法,进行拦截器的添加

			@Override
			public void addInterceptors(InterceptorRegistry registry) {
				// 多个拦截器组成一个拦截器链
				// addPathPatterns 用于添加拦截规则
				// excludePathPatterns 用户排除拦截
				registry.addInterceptor(new MyInterceptor1()).addPathPatterns("/**").excludePathPatterns("/test/**");
				registry.addInterceptor(new MyInterceptor2()).addPathPatterns("/**").excludePathPatterns("/test/**");
				super.addInterceptors(registry);
			}

--------------------------------
Spring-boot 静态资源			|
--------------------------------
	# 自动配置类的: addResourceHandlers 方法中定义了以下静态资源的自动配置
	# 类路径文件
		* /static 
		* /public
		* /resources
		* /META-INF/resources
	# 以上文件夹中的静态文件,直接可以映射为静态文件夹目录,来进行访问
	
	# webjar,的静态资源映射
		* webjar,就是jar中有jar.
		* /META-INF/resources/webjars/ 下的静态文件映射为: /web/jar/**						*/

	# 静态资源访问失败

		* 解决方案一
			* 自定义配置类,实现 WebMvcConfigurerAdapter ,覆写方法
			@Override
			public void addResourceHandlers(ResourceHandlerRegistry registry) {
				registry.addResourceHandler("/static/**").addResourceLocations("classpath:/static/");
				super.addResourceHandlers(registry);
			}
			* 该方法还可以访问外部文件(使用"file:"开头))
				registry.addResourceHandler("/upload/**").addResourceLocations("file:"+ TaleUtils.getUplodFilePath()+"upload/");

		* 解决方案二
			* 在application.properties 添加配置
				spring.mvc.static-path-pattern=/static/**								*/
				*  '注意,后面不能有空格,这里完全是为了处理掉 /** Java的注释冲突'

--------------------------------
Spring-boot 视图映射			|
--------------------------------
	# 自定义配置类,实现 WebMvcConfigurerAdapter
	# 覆写方法
		public void addViewControllers(ViewControllerRegistry registry) {
			registry.addViewController("/demo.html").setViewName("/demo.jsp");
		}

--------------------------------
Spring-boot View				|
--------------------------------
	# Jsp在内嵌的Servlet容器上运行也许有一些问题(官方不建议)
	# 所以建议以其他的模版引擎来运行
	# 关于 视图层技术单独列笔记讲


--------------------------------
Spring-boot 添加context-param	|
--------------------------------
	# 注册 ServletContextInitializer
	1,全局的context-param
		@Bean
		public ServletContextInitializer initializer() {
			return new ServletContextInitializer() {
				public void onStartup(ServletContext servletContext) throws ServletException {
					servletContext.setInitParameter("ClassLoaderLeakPreventor.threadWaitMs", "1000");
				}
			};
		}

--------------------------------
Spring-boot 跨域支持			|
--------------------------------
	@Configuration  
	public class CorsConfig extends WebMvcConfigurerAdapter {  
		@Override  
		public void addCorsMappings(CorsRegistry registry) {  
			registry.addMapping("/**")  
					.allowedOrigins("*")  
					.allowCredentials(true)  
					.allowedMethods("GET", "POST", "DELETE", "PUT")  
					.maxAge(3600);  
		}  
	}  

--------------------------------
Spring-boot 注册WEB三大组件		|
--------------------------------
	# 当使用嵌入式的Servlet容器,需要使用三个组件的时候,有三种方式

	# 把Servlet,Filter,Listenner 声明为Spring Bean 而达到注册的效果
		@Bean
		public XxxServlet xxxServlet(){
			return new XxxServlet();
		}
		@Bean
		public XxxFilter xxxFilter(){
			return new XxxFilter();
		}
		@Bean
		public XxxListenner xxxListennr(){
			return new XxxListenner();
		}

	# 注册对应的 RegistrationBean
		* 他们可以进行'参数'设置等操作
		* ServletRegistrationBean
		* FilterRegistrationBean
		* ServletListenerRegistrationBean
		
		@Configuration
		public class ServletConfig {
			/**
			 * 注册Servlet
			 * @return
			 */
			@Bean
			public ServletRegistrationBean myServlet(){
				return new ServletRegistrationBean(new TestServlet(),"/test/test");
			}
			
			/**
			 * 注册Filter
			 * @return
			 */
			@Bean
			public FilterRegistrationBean myFilter(){
				FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
				filterRegistrationBean.setFilter(new TestFileter());
				filterRegistrationBean.setOrder(2);
				return filterRegistrationBean;
			}
			
			/**
			 * 注册Listnner
			 * @return
			 */
			@Bean
			public ServletListenerRegistrationBean<MyListenner> myListenner(){
				return new ServletListenerRegistrationBean(new MyListenner());
			}
		}
	
	# 使用servlet3.0的注解
		@ServletComponentScan
			# 在 SpringBootApplication 上使用@ServletComponentScan 
			# Servlet、Filter、Listener 可以直接通过 @WebServlet、@WebFilter、@WebListener 注解自动注册，无需其他代码。