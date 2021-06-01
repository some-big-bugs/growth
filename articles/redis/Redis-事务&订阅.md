-------------------
Redis-事务			|
-------------------
	# redis的Transaction(事务)
	# redis的事务
		* redis的事务跟关系型数据库还是有一定的区别
		* 事务开启后,所有执行的命令都会按照先后顺序放入到一个队列中
		* 最后由exec(提交命令)命令原子性的提交执行
	discard
	multi
		# 标记事务开始,当执行了一条命令后.成功的添加到了队列,返回:QUEUED
	exec
		# 标记事务结束
	unwatch
	watch
-------------------
Redis-发布订阅			|
-------------------
	publish channle message
		# 把信息(message)发送给指定的频道(channle)
		# 返回:接收到message信息的订阅者数量
	subscribe channel1,channle2...
		# 订阅一个或者多个频道,那么会处于一个线程等待状态.可以及时的获取到订阅频道推送的信息
		# 收到的数据订阅格式
			(1)"message"		//固定的死格式,就是代表这是一条频道推送信息
			(2)频道名称
			(3)消息内存
	redis-cli.exe -h localhost -p6379 subscribe channle
		# windows下启动redsi客户端的时候,就订阅一个指定服务器频道
		# 不写  -h 和 -p 参数.那么默认的就是连接本机
	./redis-cli -h localhost -p6379 subscribe channle
		# linux下启动redis客户端的时候,就订阅一个指定服务器频道
		# 同上
		
		
