-----------------------
Redis-key操作			|
-----------------------
	* key的命名规则
		* 啥都可以,不能是换行,空格
		* 尽量短,也可以自己定义key的格式
			object-type:id:field
	set key value
		* 存储一个数据到内存
	mset key value key value key value..
		* 可以一次性存储多个key value
	msetnx key value key value...
		* 如果里面有一个key是已经存在的,那么不会执行任何的插入动作
	getset key newvalue
		* 给指定的key设置新的值,并且会返回原始值
	get key
		* 从内存中获得数据
	mget key key key ...
		* 从内存中获取多个指定key的数据
	exists key
		* 测试指定的Key是否存在
	del key1 key2 ...keyn
		* 删除指定的key
	keys pattern
		* 返回指定匹配模式的所有key,注意,返回的是key
		* pattern,有点像正则.但不是
	rename oldkey newkey
		* 重命名key
	dbsize
		* 返回当前数据库的key数量
	expire key seconds
		* 为key指定过期的时间
		* 设置成功后返回 1
	ttl key
		* 返回key剩余的过期秒数
		* 如果数据存在,但是没有设置过期时,间返回 -1
		* 数据已经不存在,返回 -2
	select db-index
		* 选择数据库(通过下标选择)
		* 默认是16个数据库,可以通过修改配置文件来自定义
		* 大约在84行:databases 16
	move key db-index
		* 把key从当前数据库移到指定的数据库
	flushdb
		* 清空当前数据库中的所有key
	flushall	
		* 清空所有数据库中的所有key

		
