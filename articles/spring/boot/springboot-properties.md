-------------------------------
Spring boot-配置项				|
-------------------------------
	// Server相关
		server.port=80
			# WEB监听端口

		server.context-path=/
			# WEB访问路径
		
	//日志

		logging.config=classpath:community-logback.xml
			# logback配置文件地址
	
	//静态文件
		spring.mvc.static-path-pattern=/static/**													*/
			# 用于指定静态文件的目录,允许外界直接访问

	//MyBatis
		mybatis.mapper-locations[0]=classpath*:mapper/**/Mapper.xml
			# mybatis mapper文件扫描地址

		mybatis.config-location=classpath:mybatis/mybatis-config.xml
			# mybatis配置文件地址