

-----------------------
Redis-入门				|
-----------------------

	* Redis-NoSql数据库
	* MySQL:文件存储在硬盘上
	* Redis:文件存储在内存中,但是可以序列化到硬盘
	* 介绍
		Redis(Remote Dictionary Server,远程数据服务)的缩写
		由一个意大利人开发的,内存高速缓存数据库
		使用C语言编写,数据模型为key-value
		支持丰富的数据结构
			String			//字符串,甚至是二进制数据
			List			//可以模拟队列和栈结构
			Hash			//哈希表
			Set				//无序集合
			Sorted			//有序集合
		可以持久化,保证了数据的安全
	* 缓存
		数据缓存
		页面缓存(smarty)
	* key的命名规则
		* 啥都可以,不能是换行,空格
		* 尽量短,也可以自己定义key的格式
			object-type:id:field
	* 这哥们儿还支持虚拟内存
		* 就是把硬盘的内存,腾一点出来.给计算机内存使用
	* 支持分布式
		* 一主多从
		* 一从多主
	* 支持订阅发布功能
	* 一台机器可以运行N个 Redis实例
		- 加载不同的配置文件即可.
		- 配置文件需要配置不同的端口和PID文件位置

-----------------------
Redis-安装				|
-----------------------
	Linux(CentOS6.4)
		* 把文件包通过FTP上传至Linux服务器,保存在某个目录下
		* 安装编译依赖
			yum -y install gcc tcl
		* tar zxvf redis-x.x.xx.tar.gz	//解压完成后会生成一个文件夹目录、
		* cd redis-x.x.xx				//进入解压后的文件夹目录
			* 设置安装目录 ./configure --prefix=/usr/local/redis
		* make							//执行执行编译指令就
		* make install					//编译没有问题,执行安装


		* cd src						//进入src目录
			* redis-check-aof			
			* redis-check-dump
			* redis-cli					//终端脚本
			* redis-sentinel
			* redis-server				//启动redis服务的脚本
		* mkdir /usr/local/redis		//创建reids运行目录
		* cp redis-cli redis-server /usr/local/redis	//拷贝文件到运行目录
		* cd ..							//退回到解压文件目录
			* redis.conf				//配置文件
		* cp redis.conf /usr/local/redis//拷贝配置文件到redis的运行目录
		* cd /usr/loacl/redis			//进入redis运行目录
		* ./redis-server				//启动服务,前台方式
		//后台启动redis服务
		* vim redis.conf				//打开配置文件
			* daemonize no				//17行代码大约,no:代表前台启动,yes代表后台启动
		* ./redis-server redis.conf		//重新载入配置文件,其实后台已经启动了redis服务
		* ps -A | grep redis			//查看redis进程是否启动
		//登录redis服务
		* ./redis-cli
		//登录远程的redis服务器
		./redis-cli -h localhost -p 6379 -a password
			* ip默认为本地
			* 端口默认是6379
	Windows
		* 直接解压就OK
		* 在解压目录手动的创建配置文件:redis.conf
		* 开启服务
			* 直接双击redis-server.exe 启动服务
			* 加载配置文件启动服务:redis-server.exe redis.conf
		* 双击redis-cli 启动客户端

-----------------------
Redis-支持的数据类型	|
-----------------------	
	Strings		(Binary-safe strings)
	Lists		(Lists of binary-safe strings)
	Sets		(Sets of binary-safr strings)
	Sorted sets	(Sorted sets of binary-safe strings)
	Hash

-----------------------
Redis-常用命令			|
-----------------------	
	redis开启服务
		redis-server ./redis.conf

	
	redis关闭服务
		1,redis-cli shutdown		//通过客户端关闭服务
		2,killall redis-server		//通过结束进程关闭服务
		

	1,把Redis的安装软件(源码)下载到本地,上传到Linux服务器
	2,解压tar.gz文件
		* tar -zxvf redis.tar.gz
		* 解压完成后,会在当前的目录生成一个同名文件夹
	3,进入解压目录,进行编译源码,并且安装文件
		* make
		* 如果没安装C++的编译环境,那么需要先进行安装
	4,编译成功后,进入src目录
		* 一大波东西,无视不管,最重要的几个文件(绿色的)
	5,把几个绿色的文件复制到指定的目标目录
		1,redis-benchmark		//压力测试工具
		2,redis-check-aof		//检查aof文件完整性的工具
		3,redis-check-dump		//检查数据文件的完整性工具
		4,redis-sentinel		//监控集群运行状态
		5,redis-server			//服务端
		6,redis-cli				//客户端
	6,在目标目录创建配置文件/建议从安装目录进行复制
		* redis-conf
	7,安装完毕,启动服务
		* ./redis-server redis.conf			//指定启动服务器加载的配置文件

-----------------------
Redis-关于版本号		|
-----------------------
	Redis的版本规则
		次版本号(第一个小数点后的数字),为偶数的,则是稳定版.为奇数的则是非稳定版
	
	# Windows 官方是不支持Windos平台的,我们现在使用的Windos平台的Redis,是微软自己从github上把源码扒下来,编译,发布,维护的!
	  所以,windos的 Redis的版本都会低于 Linux 版


-----------------------
Redis-安装-针对3.0		|
-----------------------
	1,下载
	2,解压
	3,copy 到 /usr/local/redis目录
	4,make				//编译,有可能会出现问题.依赖缺省.就要先装依赖
	5,make install		//安装
	6,OK~


-----------------------
Redis-后台运行			|
-----------------------
	1,在启动参数后添加 & 号
		./redis-server ./redis.conf &

	2,修改配置文件
		daemonize no				
		* 17行代码大约,no:代表前台启动,yes代表后台启动
		redis-server ./redis.conf	


-----------------------
Redis-配置文件			|
-----------------------
		port 6379						//指定端口
		pidfile	/var/run/redis.pid		//指定PID文件地址
		daemonize no					//是否在后台启动
		requirepass	xxx					//设置Redis密码
		slaveof ip port					//设置主从

		
	
		