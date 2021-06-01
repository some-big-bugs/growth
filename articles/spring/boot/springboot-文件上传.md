---------------------------
Spring-boot 文件上传		|
---------------------------
	# Spring Boot采用Servlet 3 javax.servlet.http.Part API来支持文件上传。 
	# 默认情况下， Spring Boot配置Spring MVC在单个请求中每个文件最大1Mb， 最多10Mb的文件数据。 
	# 你可以覆盖那些值， 也可以设置临时文件存储的位置（ 比如， 存储到 /tmp 文件夹下） 及传递数据刷新到磁盘的阀值（ 通过使用MultipartProperties类暴露的属性） 。
	# 如果你需要设置文件不受限制， 例如， 可以设置 multipart.maxFileSize 属性值为 -1 。
	# 当你想要接收部分（ multipart） 编码文件数据作为Spring MVC控制器（ controller） 处理方法中被 @RequestParam 注解的
	# MultipartFile类型的参数时， multipart支持就非常有用了。
	# 具体参考 MultipartAutoConfiguration 源码

---------------------------
Spring-boot 单个文件		|
---------------------------
	1,配置上传支持

		@Bean  
		public MultipartConfigElement multipartConfigElement() {  
			MultipartConfigFactory factory = new MultipartConfigFactory();  
			factory.setMaxFileSize("128KB");  
			factory.setMaxRequestSize("128KB");  
			return factory.createMultipartConfig();  
		}  
		
	2,Controller
		 @RequestParam("file") MultipartFile file	//接收文件对象
		 @RequestParam("name") String name			//正常接收其他的参数
		
	

---------------------------
Spring-boot 批量上传		|
---------------------------
	1,配置
		* 同上不解释
	
	2,Controller
		* 需要传入request对象,并且强制转换为 MultipartHttpServletRequest  
		* 通过 getFiles(String name); 来获取文件集合对象
		* 第一种方式
			List<MultipartFile> files = ((MultipartHttpServletRequest)request).getFiles("file");  
		
		2,第二种方式
			 @RequestParam("files") MultipartFile[] file