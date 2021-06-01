# 配置文件详解
	* redis.conf
	* 复制到Server目录	
	
	
	daemonize no
		* Redis服务是否后台启动,默认值为:no也就是后台启动.
		* 修改为:yes 那么Redis会以后台启动的方式启动服务
	port 6379
		* 指定Redis监听的端口.默认为:6379
	pidfile /var/run/redis.pid
		* 当Redis在后台的运行的时候,默认会把pid文件放在:/var/run/redis.pid
		* 可以配置为其他的地址,当运行多个redis服务的时候,需要指定不同管道pid,文件,和端口
	