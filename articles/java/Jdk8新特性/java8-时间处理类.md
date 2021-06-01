----------------------------
JAVA8新特性-前言			|
----------------------------
	# 原来时间API存在的问题
		* 原来的API
			Calendar
			Date
			SimpleDateFormat

		* 不好用
		* 而且都不是线程安全的

	# 新的类库
		java.time		
			* 时间API
			LocalDate			//本地日期
			LocalTime			//本地时间
			LocalDateTime		//本地日期时间
			Instant				//时间戳
			OffsetDateTime		//日期时间偏移
			OffsetTime			//时间偏移
			Period				//计算两个日期之间的间隔
			Duration			//计算两个时间之间的间隔

		java.time.zone
			* 时区API
			ZoneId				//时区
			ZoneOffset			//时区偏移
				ZoneOffset.ofHours(8);
		java.time.temporal
			* 时间的运算API
			
		java.time.format
			* 格式化API
			DateTimeFormatter
			DateTimeFormatterBuilder
			DecimalStyle
		
		TemporalAdjuster
			* 时间校正器,其实就是调整时间
		TemporalAdjusters


----------------------------
JAVA8新特性-LocalDate		|
----------------------------
	* LocalDate类使用ISO日历表示年月日.
	* 常用方法
		LocalDate LocalDate.now();
			* 获取系统当前日期

		LocalDate LocalDate.of(int year,int month,int dayOfMonth);
			* 根据指定日期创建LocalDate对象

		int getYear();
			* 返回日期中的年份

		Month month = localDate.getMonth();
			* 返回日期中的月份,返回值是 Month 对象

		int getMonthValue();
			* 返回日期中的月份,返回值是 int 

		int getDayOfMonth();
			* 返回月份中的日

		boolean isLeapYear();
			* 是否是闰年

		DayOfWeek dayOfWeek = localDate.getDayOfWeek();
			* 获取星期几,返回值是 DayOfWeek 对象

		int get(ChronoField chronoField);
			* 根据参数(枚举),返回指定类型的数据
			* 可选值 
				ChronoField.YEAR				//年份
				ChronoField.MONTH_OF_YEAR		//月
				ChronoField.DAT_OF_MONYH		//日

		LocalDate parse(String date);
			* 根据字符串,解析成LocalDate对象
			* LocalDate date = LocalDate.parse("2014-03-18");
		
		LocalDate parse(String time,DateTimeFormatter matter);
			* 根据字符串和 matter 的格式,解析成LocalDate对象

		atTime(hour,minute,second);
			* 传入 时分秒,返回 LocalDateTime 对象
		
		atTime(time);
			* 传入 LocalTime 对象,返回 LocalDateTime 对象
		
		LocalDate withYear(int year);
			* 返回一个新的 LocalDate 对象,新对象除了 year被修改,其他的都跟原来对象的属性一摸一样
		
		LocalDate withDateOfMonth(int day);
			* 同上,返回的是不同月份的 LocalDate
		
		with(ChronoField filed,int num);
			* 同上,修改的是 filed 表示的参数,num 为修改的值
		
		LocalDate plusWeeks(int num);
			* 当前时间,添加n周
		
		LocalDate minusYears(int num);
			* 当前时间减去n年
		
		plus(int num,ChronoField filed);
			* 根据 filed 表示的参数,添加 num 个单位

		LocalDate plusYears(int year);
			* 添加多少年
		LocalDate plusMonths(int month);
			* 添加多少个月
			* '还有很多添加时分秒的API,都一样'
		
		String formar(DateTimeFormatter matter);
			* 根据 DateTimeFormatter 返回被格式化后的时间字符串

	* 这个东西吧跟 Calendar 相比就是比较精确
	  Calendar 获取到的数据,月份会小一个月,我们需要加1操作.而这个就不用了

----------------------------
JAVA8新特性-LocalTime		|
----------------------------
	* LocalTime类用来表示一天中的时间
	* 大部分方法都有 LocalDate 相似,换汤不换药
	* 常用方法
		LocalTime LocalTime.now();
			* 获取系统当前时间,注意'会精确到毫秒级别'
			* 22:34:43.851
		LocalTime LocalTime.of(int hour,int minute,int second);
			* 根据指定时间创建LocalTime对象
		int getHour();
			* 返回小时
		int getMinute();
			* 返回分钟
		int getSecond();
			* 返回秒
		LocalTime parse(String time);
			* 根据字符串,解析成LocalDate对象
			* LocalTime time = LocalDate.parse("13:25:11");

		LocalTime parse(String time,DateTimeFormatter matter);
			* 根据字符串,和matter的格式,解析成LocalDate对象

		atDate(date);
			* 传递date对象.返回 LocalDateTime 对象
		
		String formar(DateTimeFormatter matter);
			* 根据 DateTimeFormatter 返回被格式化后的时间字符串
		
		LocalTime plusHours(int year);
			* 添加多少小时
		LocalTime plusMinutes(int month);
			* 添加多少分钟
			* '还有很多添加秒的API,都一样'

----------------------------
JAVA8新特性-LocalDateTime	|
----------------------------
	* LocalDateTime类表示系统的日期和时间了
	* 常用方法
		LocalDateTime LocalDateTime.now();
			* 获取系统当前时间
			* 2016-05-24T22:41:30.852
		LocalDateTime LocalDateTime.of(int year,int month,int dayOfMonth,int hour,int minute,int second);
			* 根据指定的日期和时间返回LocalDateTime对象
		int getYear();
			* 返回日期中的年份
		int getMonth();
			* 返回日期中的月份
		int getDayOfMonth();
			* 返回月份中的第几天
		int getHour();
			* 返回小时
		int getMinute();
			* 返回分钟
		int getSecond();
			* 返回秒
		LocalDateTime withDayOfMonth(int num);
			* 设置月中的天数
		LocalDateTime with(TemporalAdjusters temporalAdjusters);
			* 根据时间校正器来修改时间
		LocalDate toLocalDate();
			* 转换为 LocalDate 对象
		LocalTime toLocalTime();
			* 转换为 LocalTime 对象
		String formar(DateTimeFormatter matter);
			* 根据 DateTimeFormatter 返回被格式化后的时间字符串
		LocalDateTime plusYears(int year);
			* 添加多少年
		LocalDateTime plusMonths(int month);
			* 添加多少个月
			* '还有很多添加时分秒的API,都一样'

----------------------------
Instant						|
----------------------------
	# 时间戳(1970-1-1)工具类
	# 静态方法
		Instant now();
			* 返回当前时间戳对象
			* 默认时间获取的是UTC时区(格林威治时间)为基础的
		Instant ofEpochSecond(long sencend);
			* 获取1970-1-1 加上指定秒数后的时间戳对象

			
	# 实例方法
		OffsetDateTime atOffset(ZoneOffset zoneOffset);
			* 返回带偏移量(时差)的时间日期
		long toEpochMilli();
			* 返回时间戳对应的毫秒值
		


-------------------------------
JAVA8新特性-DateTimeFormatter	|
-------------------------------
	* DateTimeFormatter 该类用于把字符串解析为时间类
	* 常用方法
		DateTimeFormatter DateTimeFormatter.ofPatterm("yyy-MM-dd");
			* 根据指定的时间格式,返回 DateTimeFormatter 对象
		
	* 静态字段(预定义对象)
		DateTimeFormatter.BASIC_ISO_DATE;
		DateTimeFormatter.ISO_LOCAL_DATE
			* 本地日期格式


	* Demo
		DateTimeFormatter format = DateTimeFormatter.ISO_LOCAL_DATE;
		LocalDate localDate = LocalDate.parse("1993-12-09",format);
		System.out.println(localDate.toString());
		* 还可以是LocalDateTime,如果是LocalDateTime的话就必须要传递进来是有时分秒格式化的字符串
		
-------------------------------
JAVA8新特性-ZonedDateTime类		|
-------------------------------
	* ZonedDateTime类处理日期和时间与相应的时区
	* 常用方法
		ZonedDateTime.now();
			* 回去系统当前日期和时间
			* 2016-05-24T22:58:18.475+08:00[Asia/Shanghai]
		String format(DateTimeFormatter formatter);
			* 根据指定日期对象设置的格式,转换为字符串
-------------------------------
Duration						|
-------------------------------
	# 计算时间之间的间隔
	# 静态方法
		Duration between(Instant instant1,Instant instant2);
			* 计算两个时间戳之间的间隔
		
		Duration between(LocalTtime localTtime1,LocalTtime localTtime2);
			* 计算两个时间之间的间隔
	# 方法
		long toMillis();
			* 返回毫秒值
		


-------------------------------
Preiod							|
-------------------------------
	# 计算日期之间的间隔
	# 静态方法
		Preiod between(LocalDate locaDate1,LocalDate localDate2);
			* 计算两个日期之间的间隔

	# 方法
		getYears();
			* 获取年份
		getMonths();
			* 获取月份
		getDays();
			* 获取日期
		
-------------------------------
TemporalAdjusters				|
-------------------------------
	# 时间矫正器
	# 