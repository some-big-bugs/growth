# MySQL 常见优化方案

---

- [MySQL 常见优化方案](#mysql-常见优化方案)
- [1 优化数据库结构](#1-优化数据库结构)
  - [1.1 优化表的数据类型](#11-优化表的数据类型)
    - [1.1.1 选择优化的数据类型](#111-选择优化的数据类型)
    - [1.1.2 整数类型](#112-整数类型)
    - [1.1.3 实数类型](#113-实数类型)
    - [1.1.4 时间类型](#114-时间类型)
    - [1.1.5 分析已有数据](#115-分析已有数据)
  - [1.2 拆分表，提供访问效率](#12-拆分表提供访问效率)
  - [1.3 逆范式](#13-逆范式)
  - [1.4 使用冗余统计表](#14-使用冗余统计表)
  - [1.5 选择更合适的表类型](#15-选择更合适的表类型)
- [2. 定位慢SQL](#2-定位慢sql)
- [3. 通过 EXPLAIN 分析执行计划](#3-通过-explain-分析执行计划)
- [4. 索引](#4-索引)
- [5. 其他优化措施](#5-其他优化措施)
- [参考](#参考)
- [附录](#附录)
  - [show status 相关](#show-status-相关)

---

# 1 优化数据库结构

## 1.1 优化表的数据类型

`MySQL` 支持的数据类型非常多，选择正确的数据类型对获得高性能至关重要

> 数值

类型 | 大小（byte) | 有符号 | 无符号 | 用途
:---|---|---|---|---
TINYINT	| 1 |	[-128, 127] |	[0, 255] | 整数
SMALLINT | 2 | [-32768, 32767] |	[0, 65535] | 整数
MEDIUMINT	| 3 | [-8388608, 8388607] |	(0，16 777 215)	| 整数
INT或INTEGER |	4 | (-2147483648, 2147483647)	| (0，4294967295) | 整数
BIGINT |	8 | 	(-9,223,372,036,854,775,808，9 223 372 036 854 775 807)	| (0，18 446 744 073 709 551 615)	| 整数
FLOAT |	4 | 	(-3.402 823 466 E+38，-1.175 494 351 E-38)，0，(1.175 494 351 E-38，3.402 823 466 351 E+38)	 | 0，(1.175 494 351 E-38，3.402 823 466 E+38) | 	单精度 浮点数值
DOUBLE |	8 | 	(-1.797 693 134 862 315 7 E+308，-2.225 073 858 507 201 4 E-308)，0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308) | 	0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308) | 	双精度浮点数值
DECIMAL |	对DECIMAL(M,D) ，如果M>D，为M+2否则为D+2 |	依赖于M和D的值	| 依赖于M和D的值 | 指定精度小数

> 时间

类型 |	大小( byte) |	范围 |	用途
:---|:---|:---|:---
DATE |	3	| 1000-01-01/9999-12-31	| YYYY-MM-DD	日期值
TIME |	3	| '-838:59:59'/'838:59:59'	| HH:MM:SS	时间值或持续时间
YEAR |	1 |	1901/2155	YYYY	年份值
DATETIME | 	8 |	1000-01-01 00:00:00/9999-12-31 23:59:59	YYYY-MM-DD HH:MM:SS |	混合日期和时间值
TIMESTAMP |	4 |	1970-01-01 00:00:00/2038 结束时间是第 2147483647 秒，北京时间 2038-1-19 11:14:07，格林尼治时间 2038年1月19日 凌晨 03:14:07 | YYYYMMDD HHMMSS 混合日期和时间值，时间戳

> 字符

类型 |	大小(byte)	| 用途
:---|---|---
CHAR |	0-255  |	定长字符串
VARCHAR |	0-65535  |	变长字符串
TINYBLOB |	0-255  |	不超过 255 个字符的二进制字符串
TINYTEXT |	0-255  |	短文本字符串
BLOB |	0-65 535  |	二进制形式的长文本数据
TEXT |	0-65 535  |	长文本数据
MEDIUMBLOB |	0-16 777 215  |	二进制形式的中等长度文本数据
MEDIUMTEXT |	0-16 777 215  |	中等长度文本数据
LONGBLOB |	0-4 294 967 295  |	二进制形式的极大文本数据
LONGTEXT |	0-4 294 967 295  |	极大文本数据

### 1.1.1 选择优化的数据类型

下面介绍几个简单的原则：

**1.  更小的通常更好**

一般情况下，应该尽可能的使用可以正确存储数据的最小的数据类型。小的数据类型占用更小的磁盘，内存和 CPU 缓存，并且处理时的 CPU 周期也更小

**2.  简单就好**

简单的数据类型的操作通常需要更少的 CPU 周期。例如，整型比字符操作代价更低

**3.  尽量避免 `NULL`**

### 1.1.2 整数类型

数字包括整数和实数。存储整数可以使用： `TINYINT,SMALLINT MEDIUMINT,INT,BIGINT`. 分别使用 `8，16，24，32，64`位存储空间。

整数类型可选 UNSIGNED，表示不允许负值，这可以使正数的上限提高一倍。

有符号和无符号的性能是一样的，因此可以根据实际情况选择合适的类型。

MySQL 可以为整数类型指定宽度，例如 `INT(11),INT(1)`, 对于大多数应用没有意义，因为MySQL不会限制值的合法范围，这只对一些交互工具用来显示使用。 `INT(1)` 和 `INT(11)` 是相同的。

### 1.1.3 实数类型

实数是带有小数部分的数字。实数类型既可以存储小数也可以存储比BIGINT大的整数。

`MySQL` 既支持精确类型，也支持不精确类型

`FLOAT` 和 `DOUBLE` 支持标准的浮点运算进行近似计算

`DECIMAL` 支持精确的存储小数，

表需要使用何种数据类型，是需要根据应用来判断的。虽然应用设计的时候需要考虑字段的长度留有一定的冗余，但是不推荐让很多字段都留有大量的冗余，这样即浪费存储也浪费内存。

### 1.1.4 时间类型

需要根据时间范围选取合适的时间，这里注意下 `TIMESTAMP` 马上超过范围了

在表设计的时候，可以根据实际业务需要尽可能的使用可以正确存储数据的最小的数据类型。

### 1.1.5 分析已有数据

在线上业务已经稳定运行的时候，我们可以使用 `PROCEDURE ANALYSE()` 对当前已有应用的表类型的判断，该函数可以对数据表中的列的数据类型提出优化建议，可以根据应用的实际情况酌情考虑是否实施优化。

```sql
SELECT * FROM table_name PROCEDURE ANALYSE();  
SELECT * FROM tbl_name PROCEDURE ANALYSE(16,256)
```

输出的每一列信息都会对数据表中的列的数据类型提出优化建议。第二个例子告诉
PROCEDURE ANALYSE()不要为那些包含的值多于 16 个或者 256 字节的 ENUM 类型提出建议。如果没有这样的限制，输出信息可能很长；ENUM 定义通常很难阅读。

## 1.2 拆分表，提供访问效率

1.纵向拆分：

纵向拆分是只按照应用访问的频度，将表中经常访问的字段和不经常访问的字段拆分成两个表，经常访问的字段尽量是定长的，这样可以有效的提高表的查询和更新的效率。

2.横向拆分：

横向拆分是指按照应用的情况，有目的的将数据横向拆分成几个表或者通过分区分到多个分区中，这样可以有效的避免 Myisam 表的读取和更新导致的锁问题。

## 1.3 逆范式

数据库的规范化设计强调数据的独立性，数据应该尽可能少地冗余，因为存在过多的冗余数据，这就意味着要占用了更多的物理空间，同时也对数据的维护和一致性检查带来了问题。
但是对于查询操作很多的应用，一次查询可能需要访问多表进行，如果通过冗余纪录在相同表中，更新的代价增加不多，但是查询操作效率可以有明显提高，这种情况就可以考虑通过冗余数据来提高效率。

## 1.4 使用冗余统计表

使用create temporary table 语法，它是基于session 的表，表的数据保存在内存里面，当session 断掉后，表自然消除。
对于大表的统计分析，如果统计的数据量不大，利用insert。。。 select 将数据移到临时表中比直接在大表上做统计要效率更高。

## 1.5 选择更合适的表类型

1. 如果应用出现比较严重的锁冲突，请考虑是否更改存储引擎到 innodb，行锁机制可以有效的减少锁冲突的出现。
2. 如果应用查询操作很多，且对事务完整性要求不严格，则可以考虑使用Myisam

# 2. 定位慢SQL


# 3. 通过 EXPLAIN 分析执行计划

我们可以通过 `explain` 或者 `desc` 来获取 MySQL 的执行计划。


我们可以通过explain 或者desc 获取MySQL 如何执行 SELECT 语句的信息，包括select 语句执行过程表如何连接和连接的次序。
explain 可以知道什么时候必须为表加入索引以得到一个使用索引来寻找记录的更快的SELECT。

- select_type:	select 类型
- table: 输出结果集的表
- type: 表示表的连接类型
  - system: 当表中仅有一行时
  - ref: 使用索引进行表连接时type的值为ref
  - eq_ref: 连接没有使用索引时
  - ALL: 连接没有使用索引时
- possible_keys： 表示查询时,可以使用的索引列. 
- key：	表示使用的索引
- key_len：	索引长度
- rows：	扫描范围
- filtered 
- Extra：	执行情况的说明和描述

当表中仅有一行是type的值为system是最佳的连接类型； 当select操作中使用索引进行表连接时type的值为ref；
当select的表连接没有使用索引时，经常会看到type的值为ALL，表示对该表进行了全表扫描，这时需要考虑通过创建索引来提高表连接的效率。

# 4. 索引

**下列情况下，MySQL 不会使用已有的索引：**

1. 如果 mysql 估计使用索引比全表扫描更慢，则不使用索引。例如：如果 age 均匀分布在 1 和 100 之间，下列查询中使用索引就不是很好：
`SELECT * FROM table_name where age > 1 and age < 90`
1. 如果使用 hash 索引且 where 条件中不用＝索引列，其他> 、<、 >=、 <=均不使用索引；
2. 如果不是索引列的第一部分；
3. 如果 like 是以％开始；
4. 对 where 后边条件为字符串的一定要加引号，字符串如果为数字 mysql 会自动转为字符串，但是不使用索引。

# 5. 其他优化措施

1. 使用持久的连接数据库以避免连接开销。
2. 经常检查所有查询确实使用了必要的索引。
3. 避免在频繁更新的表上执行复杂的 SELECT 查询，以避免与锁定表有关的由于读、写冲突发生的问题。
4. 对于没有删除的行操作的 MyISAM 表，插入操作和查询操作可以并行进行，因为没有删除操作的表查询期间不会阻塞插入操作．对于确实需要执行删除操作的表，尽量在空闲时间进行批量删除操作，避免阻塞其他操作。
5. 充分利用列有默认值的事实。只有当插入的值不同于默认值时，才明确地插入值。这减少 MySQL 需要做的语法分析从而提高插入速度。
6. 对经常访问的可以重构的数据使用内存表，可以显著提高访问的效率。
7. 通过复制可以提高某些操作的性能。可以在复制服务器中分布客户的检索以均分负载。为了防止备份期间对应用的影响，可以在复制服务器上执行备份操作。
表的字段尽量不使用自增长变量，在高并发情况下该字段的自增可能对效率有比较大的影响，推荐通过应用来实现字段的自增长。

MySQL查询优化
阅读目录
1、简介
2、截取SQL语句
3、查询优化基本分析命令
4、查询优化几个方向
5、索引优化
　　5.1、索引优点：
　　5.2、索引缺点
　　5.3、索引选择
　　5.4、索引细究
6、子查询优化
7、等价谓词重写：
8、条件化简与优化
9、外连接优化
10、其他查询优化
11、博文总结
回到顶部
1、简介
     一个好的web应用，最重要的一点是有着优秀的访问性能。数据库MySQL是web应用的组成部分，也是决定其性能的重要部分。所以提升MySQL的性能至关重要。
     MySQL性能的提升可分为三部分，包括硬件、网络、软件。其中硬件、网络取决于公司的财力，需要白哗哗的银两，这里就不说啦。软件又细分为很多种，在这里我们通过MySQL的查询优化从而达到性能的提升。
     最近看了一些关于查询优化的书籍，同时也在网上看一些前辈们写的文章。
以下是自己整理借鉴关于查询优化的一些总结：
回到顶部
2、截取SQL语句
     1、全面查询日志
     2、慢查询日志
     3、二进制日志
     4、进程列表
　　SHOW FULL PROCESSLIST;
　　。。。
回到顶部
3、查询优化基本分析命令
　　1、EXPLAIN {PARTITIONS|EXTENDED}
　　2、SHOW CREATE TABLE tab;
　　3、SHOW INDEXS FROM tab;
　　4、SHOW TABLE STATUS LIKE ‘tab’;
　　5、SHOW [GLOBAL|SESSION] STATUS LIKE ‘’;
　　6、SHOW VARIABLES
　　。。。。
　　ps：我自己都感觉上面都是没任何营养的东西。下面才是真正的干货哈。
回到顶部
4、查询优化几个方向
　　1、尽量避免全文扫描，给相应字段增加索引，应用索引来查询
　　2、删除不用或者重复的索引
　　3、查询重写，等价转换（谓词、子查询、连接查询）
　　4、删除内容重复不必要的语句，精简语句
　　5、整合重复执行的语句
　　6、缓存查询结果
回到顶部
5、索引优化
回到顶部
　　5.1、索引优点：
　　　　1、保持数据的完整性
　　　　2、提高数据的查询性能
　　　　3、改进表的连接操作（jion）
　　　　4、对查询结果进行排序。没索引将会采用内部文件排序算法进行排序，效率较慢
　　　　5、简化聚合数据操作
回到顶部
　　5.2、索引缺点
　　　　1、索引需要占用一定的存储空间
　　　　2、数据插入、更新、删除时会受索引的影响，性能会降低。因为数据变更索引也需要进行更新
　　　　3、多个索引，优化器需要耗时则优选择
回到顶部
　　5.3、索引选择
　　　　1、数据量大时采用
　　　　2、数据高度重复时，不采用
　　　　3、查询取出数据大于20%，将采用全文扫描，不用索引
回到顶部
　　5.4、索引细究
　　　　资料查询：
　　　　MySQL中的InnoDB、MyISAM都是B-Tree类型索引
　　　　B-Tree包含：PRIMARY KEY, UNIQUE, INDEX, and FULLTEXT
　　　　B-Tree类型索引不支持（即字段使用以下符号时，将不采用索引）：
　　　　>, <, >=, <=, BETWEEN, !=, <>,like ‘%**’
　　　　【在此先介绍一下覆盖索引】
　　　　以我自己理解的方式介绍吧。覆盖索引并不是像主键索引、唯一索引一样真实存在，它只是对索引应用某些特定场景的一种定义【另一种理解：查询的列是索引列，因此列被索引覆盖】。它可以突破传统的限制，使用以上操作符，且依然采用索引进行查询。
　　　　因为查询的列是索引列，所以不需要读取行，只需要读取列字段数据就可以了。【例如你看一本书，需要找某一内容，刚好那内容出现在目录中，那就不用一页页翻了，直接在目录中定位到第几页查找】
　　　　如何激活覆盖索引呢？什么样才是特定场景呢？
　　　　索引字段，在select中出现就是了。
　　　　复合索引还可能有其他的特殊场景。例如，三列复合索引，仅需要在select、where、group by、order by中，任意一个地方出现一次复合索引最左边列就可以激活使用覆盖索引了。
　　　　查看：
　　　　EXPLAIN中Extra显示有Using index表示这条语句采用了覆盖索引。
　　　　结论：
　　　　不建议在查询的时候使用select*from进行查询了，应该写需要用的字段，并且增加相应的索引，以提高查询性能。
　　　　针对以上操作符实测结果：
　　　　1、以select*from形式，where中是primary key可以通杀【除like】（使用主键进行查询）；index则全不可以。
　　　　2、以select 字段a from tab where 字段a《以上操作符》形式测试，结果依然可以使用索引查询。【采用了覆盖索引】
　　　　其他索引优化方法:
　　　　1、使用索引关键字作为连接的条件
　　　　2、复合索引使用
　　　　3、索引合并or and，将涉及到的字段合并成复合索引
　　　　4、where、和group by涉及字段加索引
回到顶部
6、子查询优化
　　在from中为非相关子查询，可以上拉子查询到父层。在多表连接查询考虑连接代价再选择。
　　查询优化器对子查询一般采用嵌套执行的方式，即对父查询中的每一行，都执行一次子查询，这样子查询会执行很多次。这种执行方式效率很低。
　　子查询转化为连接查询优点：
　　1、子查询不用执行很多次
　　2、优化器可以根据信息来选择不同的方法和连接顺序
　　3、子查询的连接条件，过滤条件变成父查询的筛选条件，以提高效率。
　　优化：
　　子查询合并，若多个子查询，能合并的尽量合并。
　　子查询展开，即上拉变成多表查询（时刻保证等价变化）
　　注意：
　　子查询展开只能展开简单的查询，若子查询含有聚集函数、GROUP BY、DISTINCT，则不能上拉。
　　select * from t1 (select*from tab where id>10) as t2 where t1.age>10 and t2.age<25;
　　select*from t1,tab as t2 where t1.age>10 and t2.age<25 and t2.id>10;
　　具体步骤：
　　1、from与from合并，修改相应参数
　　2、where与where合并，用and连接
　　3、修改相应的谓词（in改=）
回到顶部
7、等价谓词重写：
　　1、BETWEEEN AND改写为 >= 、<=之类的。实测：十万条数据，重写前后时间，1.45s、0.06s
　　2、in转换多个or。字段为索引时，两个都能用到索引，or效率相对in好一点
　　3、name like ‘abc%’改写成name>=’abc’ and name<’abd’;
　　注意：百万级数据测试，name没有索引之前like比后一种查询快；给字段增加索引后，后面的快一点点，相差不大，因为两种方法在查询的时候都用到了索引。
　　。。。。
回到顶部
8、条件化简与优化
　　1、将where、having（不存在groupby和聚集函数时）、join-on条件能合并的尽量合并
　　2、删除不必要的括号，减少语法分许的or和and树层，减少cpu消耗
　　3、常量传递。a=b and b=2转换为 a=2 and b=2。尽量不使用变量a=b或a=@var
　　4、消除没用的SQL条件
　　5、where等号右边尽量不出现表达式计算；where中不要对字段进行表达式计算、函数的使用
　　6、恒等变换、不等式变换。例：测试百万级数据a>b and b>10变为a>b and a>10 and b>10优化显著
回到顶部
9、外连接优化
　　即将外连接转为内连接
　　优点：
　　1、优化处理器处理外连接比内连接步骤多且耗时
　　2、外连接消除后，优化器选择多表连接顺序有更多选择，可以择优而选
　　3、可以将筛选条件最为严格的表作为外表（连接顺序最前面，是多层循环体的外循环层），
　　可以减少不必要的I/O开销，能加快算法执行的速度。
　　on a.id=b.id与where a.id=b.id的差别，on则表进行连接，where则进行数据对比
　　注意：前提必须是结果为NULL决绝（即条件限制不要NULL数据行，语意上是内连接）
　　优化原则：
　　精简查询，连接消除，等效转换，去除多余表对象连接
　　例如：主键/唯一键作为连接条件，且中间表列只作为等值条件，可以去掉中间表连接
回到顶部
10、其他查询优化
　　1、以下将会造成放弃索引查询，采用全文扫描
　　　　1.1、where 子句中使用!=或<>操作符　　注意：主键支持。非主键不支持
　　　　1.2、避免使用or
　　　　　　经测试，并非是使用了or就一定不能使用索引，大多情况下是没用到索引，但还有少数情况是用到的，因此具体情况具体分析。
　　　　　　类似优化：
　　　　　　select * from tab name=’aa’ or name=’bb’;
　　　　　　=>
　　　　　　select * from tab name=’aa’
　　　　　　union all
　　　　　　select * from tab name=’bb’;
　　　　　　实测：
　　　　　　1、十万数据测试，没任何索引的情况下，上面比下面的查询速率快一倍。
　　　　　　2、三十万数据测试，aa与bb都是单独索引情况下，下面的查询速率比or快一点。
　　　　1.3、避免使用not in
　　　　　　not in一般不能使用索引；主键字段可以
　　　　1.4、where中尽量避免使用对null的判断
　　　　1.5、like不能前置百分号 like ‘%.com’
　　　　　　解决：
　　　　　　　　1、若必须使用%前置，且数据长度不大，例如URL，可将数据翻转存入数据库，再来查。LIKE REVERSE‘%.com’;
　　　　　　　　2、使用覆盖索引
 
　　　　1.6、使用索引字段作为条件的时候，假若是复合索引，则应该使用索引最左边前缀的字段名
　　2、将exists代替in
　　　　select num from a where num in(select num from b)
　　　　select num from a where exists(select 1 from b where num=a.num)
　　　　一百万条数据，筛选59417条数据用时6.65s、4.18s。没做其他优化，仅仅只是将exists替换in。
　　3、字段定义是字符串，查询时没带引号，不会用索引，将会进行全文扫描。
　　【以下是摘抄于半夜乱弹琴博文http://www.cnblogs.com/lingiu/p/3414134.html，本人没进行相应的测试】
　　4、尽量使用表变量来代替临时表
　　5、避免频繁创建和删除临时表，以减少系统表资源的消耗
　　6、如果使用到了临时表，在存储过程的最后务必将所有的临时表显式删除，先 truncate table ，然后 drop table ，这样可以避免系统表的较长时间锁定
　　7、尽量避免使用游标，因为游标的效率较差，如果游标操作的数据超过1万行，那么就应该考虑改写
　　8、大数据量，若数据量过大，应该考虑相应需求是否合理。
　　9、尽量避免大事务操作，提高系统并发能力。
　　。。。。。

# 参考

1. 高性能 MySQL
2. https://dev.mysql.com/doc/refman/5.7/en/

# 附录

## show status 相关

| Variable_name                                 | Value                                            |
| --------------------------------------------- | :----------------------------------------------- |
| Aborted_clients                               | 0                                                |
| Aborted_connects                              | 0                                                |
| Binlog_cache_disk_use                         | 0                                                |
| Binlog_cache_use                              | 0                                                |
| Binlog_stmt_cache_disk_use                    | 0                                                |
| Binlog_stmt_cache_use                         | 0                                                |
| Bytes_received                                | 185                                              |
| Bytes_sent                                    | 645                                              |
| Com_admin_commands                            | 0                                                |
| Com_assign_to_keycache                        | 0                                                |
| Com_alter_db                                  | 0                                                |
| Com_alter_db_upgrade                          | 0                                                |
| Com_alter_event                               | 0                                                |
| Com_alter_function                            | 0                                                |
| Com_alter_instance                            | 0                                                |
| Com_alter_procedure                           | 0                                                |
| Com_alter_server                              | 0                                                |
| Com_alter_table                               | 0                                                |
| Com_alter_tablespace                          | 0                                                |
| Com_alter_user                                | 0                                                |
| Com_analyze                                   | 0                                                |
| Com_begin                                     | 0                                                |
| Com_binlog                                    | 0                                                |
| Com_call_procedure                            | 0                                                |
| Com_change_db                                 | 0                                                |
| Com_change_master                             | 0                                                |
| Com_change_repl_filter                        | 0                                                |
| Com_check                                     | 0                                                |
| Com_checksum                                  | 0                                                |
| Com_commit                                    | 0                                                |
| Com_create_db                                 | 0                                                |
| Com_create_event                              | 0                                                |
| Com_create_function                           | 0                                                |
| Com_create_index                              | 0                                                |
| Com_create_procedure                          | 0                                                |
| Com_create_server                             | 0                                                |
| Com_create_table                              | 0                                                |
| Com_create_trigger                            | 0                                                |
| Com_create_udf                                | 0                                                |
| Com_create_user                               | 0                                                |
| Com_create_view                               | 0                                                |
| Com_dealloc_sql                               | 0                                                |
| Com_delete                                    | 0                                                |
| Com_delete_multi                              | 0                                                |
| Com_do                                        | 0                                                |
| Com_drop_db                                   | 0                                                |
| Com_drop_event                                | 0                                                |
| Com_drop_function                             | 0                                                |
| Com_drop_index                                | 0                                                |
| Com_drop_procedure                            | 0                                                |
| Com_drop_server                               | 0                                                |
| Com_drop_table                                | 0                                                |
| Com_drop_trigger                              | 0                                                |
| Com_drop_user                                 | 0                                                |
| Com_drop_view                                 | 0                                                |
| Com_empty_query                               | 0                                                |
| Com_execute_sql                               | 0                                                |
| Com_explain_other                             | 0                                                |
| Com_flush                                     | 0                                                |
| Com_get_diagnostics                           | 0                                                |
| Com_grant                                     | 0                                                |
| Com_ha_close                                  | 0                                                |
| Com_ha_open                                   | 0                                                |
| Com_ha_read                                   | 0                                                |
| Com_help                                      | 0                                                |
| Com_insert                                    | 0                                                |
| Com_insert_select                             | 0                                                |
| Com_install_plugin                            | 0                                                |
| Com_kill                                      | 0                                                |
| Com_load                                      | 0                                                |
| Com_lock_tables                               | 0                                                |
| Com_optimize                                  | 0                                                |
| Com_preload_keys                              | 0                                                |
| Com_prepare_sql                               | 0                                                |
| Com_purge                                     | 0                                                |
| Com_purge_before_date                         | 0                                                |
| Com_release_savepoint                         | 0                                                |
| Com_rename_table                              | 0                                                |
| Com_rename_user                               | 0                                                |
| Com_repair                                    | 0                                                |
| Com_replace                                   | 0                                                |
| Com_replace_select                            | 0                                                |
| Com_reset                                     | 0                                                |
| Com_resignal                                  | 0                                                |
| Com_revoke                                    | 0                                                |
| Com_revoke_all                                | 0                                                |
| Com_rollback                                  | 0                                                |
| Com_rollback_to_savepoint                     | 0                                                |
| Com_savepoint                                 | 0                                                |
| Com_select                                    | 2                                                |
| Com_set_option                                | 0                                                |
| Com_signal                                    | 0                                                |
| Com_show_binlog_events                        | 0                                                |
| Com_show_binlogs                              | 0                                                |
| Com_show_charsets                             | 0                                                |
| Com_show_collations                           | 0                                                |
| Com_show_create_db                            | 0                                                |
| Com_show_create_event                         | 0                                                |
| Com_show_create_func                          | 0                                                |
| Com_show_create_proc                          | 0                                                |
| Com_show_create_table                         | 0                                                |
| Com_show_create_trigger                       | 0                                                |
| Com_show_databases                            | 1                                                |
| Com_show_engine_logs                          | 0                                                |
| Com_show_engine_mutex                         | 0                                                |
| Com_show_engine_status                        | 0                                                |
| Com_show_events                               | 0                                                |
| Com_show_errors                               | 0                                                |
| Com_show_fields                               | 0                                                |
| Com_show_function_code                        | 0                                                |
| Com_show_function_status                      | 0                                                |
| Com_show_grants                               | 0                                                |
| Com_show_keys                                 | 0                                                |
| Com_show_master_status                        | 0                                                |
| Com_show_open_tables                          | 0                                                |
| Com_show_plugins                              | 0                                                |
| Com_show_privileges                           | 0                                                |
| Com_show_procedure_code                       | 0                                                |
| Com_show_procedure_status                     | 0                                                |
| Com_show_processlist                          | 0                                                |
| Com_show_profile                              | 0                                                |
| Com_show_profiles                             | 0                                                |
| Com_show_relaylog_events                      | 0                                                |
| Com_show_slave_hosts                          | 0                                                |
| Com_show_slave_status                         | 0                                                |
| Com_show_status                               | 1                                                |
| Com_show_storage_engines                      | 0                                                |
| Com_show_table_status                         | 0                                                |
| Com_show_tables                               | 0                                                |
| Com_show_triggers                             | 0                                                |
| Com_show_variables                            | 0                                                |
| Com_show_warnings                             | 0                                                |
| Com_show_create_user                          | 0                                                |
| Com_shutdown                                  | 0                                                |
| Com_slave_start                               | 0                                                |
| Com_slave_stop                                | 0                                                |
| Com_group_replication_start                   | 0                                                |
| Com_group_replication_stop                    | 0                                                |
| Com_stmt_execute                              | 0                                                |
| Com_stmt_close                                | 0                                                |
| Com_stmt_fetch                                | 0                                                |
| Com_stmt_prepare                              | 0                                                |
| Com_stmt_reset                                | 0                                                |
| Com_stmt_send_long_data                       | 0                                                |
| Com_truncate                                  | 0                                                |
| Com_uninstall_plugin                          | 0                                                |
| Com_unlock_tables                             | 0                                                |
| Com_update                                    | 0                                                |
| Com_update_multi                              | 0                                                |
| Com_xa_commit                                 | 0                                                |
| Com_xa_end                                    | 0                                                |
| Com_xa_prepare                                | 0                                                |
| Com_xa_recover                                | 0                                                |
| Com_xa_rollback                               | 0                                                |
| Com_xa_start                                  | 0                                                |
| Com_stmt_reprepare                            | 0                                                |
| Compression                                   | OFF                                              |
| Connection_errors_accept                      | 0                                                |
| Connection_errors_internal                    | 0                                                |
| Connection_errors_max_connections             | 0                                                |
| Connection_errors_peer_address                | 0                                                |
| Connection_errors_select                      | 0                                                |
| Connection_errors_tcpwrap                     | 0                                                |
| Connections                                   | 4                                                |
| Created_tmp_disk_tables                       | 0                                                |
| Created_tmp_files                             | 5                                                |
| Created_tmp_tables                            | 1                                                |
| Delayed_errors                                | 0                                                |
| Delayed_insert_threads                        | 0                                                |
| Delayed_writes                                | 0                                                |
| Flush_commands                                | 1                                                |
| Handler_commit                                | 0                                                |
| Handler_delete                                | 0                                                |
| Handler_discover                              | 0                                                |
| Handler_external_lock                         | 0                                                |
| Handler_mrr_init                              | 0                                                |
| Handler_prepare                               | 0                                                |
| Handler_read_first                            | 0                                                |
| Handler_read_key                              | 0                                                |
| Handler_read_last                             | 0                                                |
| Handler_read_next                             | 0                                                |
| Handler_read_prev                             | 0                                                |
| Handler_read_rnd                              | 0                                                |
| Handler_read_rnd_next                         | 21                                               |
| Handler_rollback                              | 0                                                |
| Handler_savepoint                             | 0                                                |
| Handler_savepoint_rollback                    | 0                                                |
| Handler_update                                | 0                                                |
| Handler_write                                 | 20                                               |
| Innodb_buffer_pool_dump_status                | Dumping of buffer pool not started               |
| Innodb_buffer_pool_load_status                | Buffer pool(s) load completed at 201126 20:19:10 |
| Innodb_buffer_pool_resize_status              |
| Innodb_buffer_pool_pages_data                 | 490                                              |
| Innodb_buffer_pool_bytes_data                 | 8028160                                          |
| Innodb_buffer_pool_pages_dirty                | 0                                                |
| Innodb_buffer_pool_bytes_dirty                | 0                                                |
| Innodb_buffer_pool_pages_flushed              | 37                                               |
| Innodb_buffer_pool_pages_free                 | 7697                                             |
| Innodb_buffer_pool_pages_misc                 | 4                                                |
| Innodb_buffer_pool_pages_total                | 8191                                             |
| Innodb_buffer_pool_read_ahead_rnd             | 0                                                |
| Innodb_buffer_pool_read_ahead                 | 0                                                |
| Innodb_buffer_pool_read_ahead_evicted         | 0                                                |
| Innodb_buffer_pool_read_requests              | 5388                                             |
| Innodb_buffer_pool_reads                      | 456                                              |
| Innodb_buffer_pool_wait_free                  | 0                                                |
| Innodb_buffer_pool_write_requests             | 515                                              |
| Innodb_data_fsyncs                            | 7                                                |
| Innodb_data_pending_fsyncs                    | 0                                                |
| Innodb_data_pending_reads                     | 0                                                |
| Innodb_data_pending_writes                    | 0                                                |
| Innodb_data_read                              | 7541248                                          |
| Innodb_data_reads                             | 526                                              |
| Innodb_data_writes                            | 54                                               |
| Innodb_data_written                           | 641024                                           |
| Innodb_dblwr_pages_written                    | 2                                                |
| Innodb_dblwr_writes                           | 1                                                |
| Innodb_log_waits                              | 0                                                |
| Innodb_log_write_requests                     | 0                                                |
| Innodb_log_writes                             | 2                                                |
| Innodb_os_log_fsyncs                          | 4                                                |
| Innodb_os_log_pending_fsyncs                  | 0                                                |
| Innodb_os_log_pending_writes                  | 0                                                |
| Innodb_os_log_written                         | 1024                                             |
| Innodb_page_size                              | 16384                                            |
| Innodb_pages_created                          | 35                                               |
| Innodb_pages_read                             | 455                                              |
| Innodb_pages_written                          | 37                                               |
| Innodb_row_lock_current_waits                 | 0                                                |
| Innodb_row_lock_time                          | 0                                                |
| Innodb_row_lock_time_avg                      | 0                                                |
| Innodb_row_lock_time_max                      | 0                                                |
| Innodb_row_lock_waits                         | 0                                                |
| Innodb_rows_deleted                           | 0                                                |
| Innodb_rows_inserted                          | 0                                                |
| Innodb_rows_read                              | 8                                                |
| Innodb_rows_updated                           | 0                                                |
| Innodb_num_open_files                         | 62                                               |
| Innodb_truncated_status_writes                | 0                                                |
| Innodb_available_undo_logs                    | 128                                              |
| Key_blocks_not_flushed                        | 0                                                |
| Key_blocks_unused                             | 6695                                             |
| Key_blocks_used                               | 3                                                |
| Key_read_requests                             | 6                                                |
| Key_reads                                     | 3                                                |
| Key_write_requests                            | 0                                                |
| Key_writes                                    | 0                                                |
| Last_query_cost                               | 0                                                |
| Last_query_partial_plans                      | 0                                                |
| Locked_connects                               | 0                                                |
| Max_execution_time_exceeded                   | 0                                                |
| Max_execution_time_set                        | 0                                                |
| Max_execution_time_set_failed                 | 0                                                |
| Max_used_connections                          | 1                                                |
| Max_used_connections_time                     | 44161.9190625                                    |
| Not_flushed_delayed_rows                      | 0                                                |
| Ongoing_anonymous_transaction_count           | 0                                                |
| Open_files                                    | 14                                               |
| Open_streams                                  | 0                                                |
| Open_table_definitions                        | 791                                              |
| Open_tables                                   | 99                                               |
| Opened_files                                  | 939                                              |
| Opened_table_definitions                      | 0                                                |
| Opened_tables                                 | 0                                                |
| Performance_schema_accounts_lost              | 0                                                |
| Performance_schema_cond_classes_lost          | 0                                                |
| Performance_schema_cond_instances_lost        | 0                                                |
| Performance_schema_digest_lost                | 0                                                |
| Performance_schema_file_classes_lost          | 0                                                |
| Performance_schema_file_handles_lost          | 0                                                |
| Performance_schema_file_instances_lost        | 0                                                |
| Performance_schema_hosts_lost                 | 0                                                |
| Performance_schema_index_stat_lost            | 0                                                |
| Performance_schema_locker_lost                | 0                                                |
| Performance_schema_memory_classes_lost        | 0                                                |
| Performance_schema_metadata_lock_lost         | 0                                                |
| Performance_schema_mutex_classes_lost         | 0                                                |
| Performance_schema_mutex_instances_lost       | 0                                                |
| Performance_schema_nested_statement_lost      | 0                                                |
| Performance_schema_prepared_statements_lost   | 0                                                |
| Performance_schema_program_lost               | 0                                                |
| Performance_schema_rwlock_classes_lost        | 0                                                |
| Performance_schema_rwlock_instances_lost      | 0                                                |
| Performance_schema_session_connect_attrs_lost | 0                                                |
| Performance_schema_socket_classes_lost        | 0                                                |
| Performance_schema_socket_instances_lost      | 0                                                |
| Performance_schema_stage_classes_lost         | 0                                                |
| Performance_schema_statement_classes_lost     | 0                                                |
| Performance_schema_table_handles_lost         | 0                                                |
| Performance_schema_table_instances_lost       | 0                                                |
| Performance_schema_table_lock_stat_lost       | 0                                                |
| Performance_schema_thread_classes_lost        | 0                                                |
| Performance_schema_thread_instances_lost      | 0                                                |
| Performance_schema_users_lost                 | 0                                                |
| Prepared_stmt_count                           | 0                                                |
| Qcache_free_blocks                            | 1                                                |
| Qcache_free_memory                            | 1031832                                          |
| Qcache_hits                                   | 0                                                |
| Qcache_inserts                                | 0                                                |
| Qcache_lowmem_prunes                          | 0                                                |
| Qcache_not_cached                             | 2                                                |
| Qcache_queries_in_cache                       | 0                                                |
| Qcache_total_blocks                           | 1                                                |
| Queries                                       | 6                                                |
| Questions                                     | 4                                                |
| Select_full_join                              | 0                                                |
| Select_full_range_join                        | 0                                                |
| Select_range                                  | 0                                                |
| Select_range_check                            | 0                                                |
| Select_scan                                   | 1                                                |
| Slave_open_temp_tables                        | 0                                                |
| Slow_launch_threads                           | 0                                                |
| Slow_queries                                  | 0                                                |
| Sort_merge_passes                             | 0                                                |
| Sort_range                                    | 0                                                |
| Sort_rows                                     | 0                                                |
| Sort_scan                                     | 0                                                |
| Ssl_accept_renegotiates                       | 0                                                |
| Ssl_accepts                                   | 0                                                |
| Ssl_callback_cache_hits                       | 0                                                |
| Ssl_cipher                                    |
| Ssl_cipher_list                               |
| Ssl_client_connects                           | 0                                                |
| Ssl_connect_renegotiates                      | 0                                                |
| Ssl_ctx_verify_depth                          | 0                                                |
| Ssl_ctx_verify_mode                           | 0                                                |
| Ssl_default_timeout                           | 0                                                |
| Ssl_finished_accepts                          | 0                                                |
| Ssl_finished_connects                         | 0                                                |
| Ssl_server_not_after                          |
| Ssl_server_not_before                         |
| Ssl_session_cache_hits                        | 0                                                |
| Ssl_session_cache_misses                      | 0                                                |
| Ssl_session_cache_mode                        | NONE                                             |
| Ssl_session_cache_overflows                   | 0                                                |
| Ssl_session_cache_size                        | 0                                                |
| Ssl_session_cache_timeouts                    | 0                                                |
| Ssl_sessions_reused                           | 0                                                |
| Ssl_used_session_cache_entries                | 0                                                |
| Ssl_verify_depth                              | 0                                                |
| Ssl_verify_mode                               | 0                                                |
| Ssl_version                                   |
| Table_locks_immediate                         | 99                                               |
| Table_locks_waited                            | 0                                                |
| Table_open_cache_hits                         | 0                                                |
| Table_open_cache_misses                       | 0                                                |
| Table_open_cache_overflows                    | 0                                                |
| Tc_log_max_pages_used                         | 0                                                |
| Tc_log_page_size                              | 0                                                |
| Tc_log_page_waits                             | 0                                                |
| Threads_cached                                | 0                                                |
| Threads_connected                             | 1                                                |
| Threads_created                               | 1                                                |
| Threads_running                               | 1                                                |
| Uptime                                        | 6268                                             |
| Uptime_since_flush_status                     | 6268                                             |

