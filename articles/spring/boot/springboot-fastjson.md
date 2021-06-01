-----------------------------
Spring boot 使用 fastjson	 |
-----------------------------
	# spring 默认的json处理框架是 jackson
	# fastjson 是阿里巴巴的一个高性能json序列化框架
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>fastjson</artifactId>
			<version>1.2.24</version>
		</dependency>
	
-----------------------------
Spring boot 方法一			 |
-----------------------------
	1,启动类继承 WebMvcConfigurerAdapter
	2,覆盖方法 configureMessageConverters

	# 代码
		@SpringBootApplication
		public class Application extends WebMvcConfigurerAdapter {
			public static void main(String[] args){
				SpringApplication.run(Application.class,args);
			}
			
			@Override
			public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
				super.configureMessageConverters(converters);
				//定义 Converter 消息转换器
				FastJsonHttpMessageConverter fastJsonHttpMessageConverter = new FastJsonHttpMessageConverter();
				//定义 消息转换器配置对象
				FastJsonConfig fastJsonConfig = new FastJsonConfig();
				//进行配置操作
				fastJsonConfig.setSerializerFeatures(SerializerFeature.PrettyFormat);
				//配置 converter 消息转换器
				fastJsonHttpMessageConverter.setFastJsonConfig(fastJsonConfig);
				//把converter 添加到 converters 中
				converters.add(fastJsonHttpMessageConverter);
			}
		}


-----------------------------
Spring boot 方法二			 |
-----------------------------
	# 以组件的形式把 fastjson 的消息转换器.加入IOC
	# 代码
		/**
		 * Created by KevinBlandy on 2017/2/25 16:47
		 */
		@Configuration
		public class HttpMessageConverterConfiguration {
			/**
			 * FastJsonpHttpMessageConverter4
			 * @return
			 */
			@Bean
			public HttpMessageConverters httpMessageConverter(){
				FastJsonpHttpMessageConverter4 fastJsonpHttpMessageConverter4 = new FastJsonpHttpMessageConverter4();
				fastJsonpHttpMessageConverter4.setSupportedMediaTypes(Arrays.asList(MediaType.APPLICATION_JSON_UTF8));
				FastJsonConfig fastJsonConfig = new FastJsonConfig();
				fastJsonConfig.setCharset(StandardCharsets.UTF_8);
				fastJsonConfig.setSerializerFeatures(
						SerializerFeature.PrettyFormat,				//格式化
						SerializerFeature.WriteMapNullValue,		//输出null字段
						SerializerFeature.QuoteFieldNames,			//使用双引号
						SerializerFeature.WriteNullListAsEmpty);	//把null集合/数组输出为[]
				fastJsonConfig.setDateFormat("yyyy-MM-dd HH:mm:ss");
				fastJsonpHttpMessageConverter4.setFastJsonConfig(fastJsonConfig);
				return new HttpMessageConverters(fastJsonpHttpMessageConverter4);
			}
			
			/**
			 * 跨域支持
			 * @return
			 */
			@Bean
			public FastJsonpResponseBodyAdvice fastJsonpResponseBodyAdvice(){
				FastJsonpResponseBodyAdvice fastJsonpResponseBodyAdvice = new FastJsonpResponseBodyAdvice("callback","jsonp");
				return fastJsonpResponseBodyAdvice;
			}
		}
		* 代码逻辑其实跟上面差不多


		
	
