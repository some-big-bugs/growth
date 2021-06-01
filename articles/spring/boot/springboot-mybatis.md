----------------------------
Spring boot MyBatis			|
----------------------------


----------------------------
Spring boot 第一种方式		|
----------------------------
	# 使用官方提供的jar,进行整合
	1,引入依赖
		
		<!-- spring boot mybatis -->
		<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>1.2.0</version>
		</dependency>
		
		<!-- mybatis分页插件 -->
		<dependency>
			<groupId>com.github.miemiedev</groupId>
			<artifactId>mybatis-paginator</artifactId>
			<version>1.2.17</version>
		</dependency>
	
	2,在 @SpringBootApplication 类上添加注解来扫描 mapper 接口
		@MapperScan("com.xx");
		* 扫描指定包下的 mapper 接口
		* value				参数是一个数组,可以扫描多个
		* annotationClass	仅仅加载添加了指定注解的类
	
	3,在properties中配置参数
		mybatis.mapper-locations[0]=classpath*:com/tedi/**/*mapper.xml																
			* mapper文件地址,可以有多个
		mybatis.config-location=classpath:mybatis/mybatis-config.xml
			* mybatis配置文件地址
		

	
	

----------------------------
Spring boot 第二种方式		|
----------------------------
	# 使用原始配置spring的方式进行整合
	# spring 是怎么整合的,就怎么整合

