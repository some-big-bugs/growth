<h1> 编写高质量代码, 改善Java程序的151个建议 </h1>

> 内容来自 https://blog.csdn.net/aishangyutian12/article/details/52699938 并对部分内容进行了删减和补充，有兴趣的同学可以看原著

### 建议1：不要在常量、变量和验证码中出现易混淆的字母
    
i、l、1；o、0等, 这些字母相近不好辨认

### 建议2：莫让常量蜕变成变量

代码运行工程中不要改变常量值

### 建议3：三元操作符的类型务必一致

不一致会导致自动类型转换，类型提升int->float->double等

### 建议4：避免带有变长参数的方法重载

变长参数的方法重载之后可能会包含原方法

### 建议5：别让null值和空值威胁到变长方法

两个都包含变长参数的重载方法，当变长参数部分空值，或者为null值时，重载方法不清楚会调用哪一个方法

### 建议6：不要让旧语法和解决方案困扰你

（Java中抛弃了C语言中的goto语法，但是还保留了该关键字，只是不进行语义处理，const关键中同样类似）。


### 建议7：避免为final变量复杂赋值

类序列化保存到磁盘上（或网络传输）的对象文件包括两部分：1、类描述信息：包括包路径、继承关系等。注意，它并不是class文件的翻版，不记录方法、构造函数、static变量等的具体实现。2、非瞬态（transient关键字）和非静态（static关键字）的实例变量值。总结：反序列化时final变量在以下情况下不会被重新赋值：1、通过构造函数为final变量赋值；2、通过方法返回值为final变量赋值；3、final修饰的属性不是基本类型

### 建议8：break万万不可忘

switch语句中，每一个case匹配完都需要使用break关键字跳出，否则会依次执行完所有的case内容。

### 建议9：用偶判断，不用奇判断；

不要使用奇判断（i%2 == 1 ? "奇数" : "偶数"），使用偶判断（i%2 == 0 ? "偶数" : "奇数"）。原因Java中的取余（%标识符）算法：测试数据输入1 2 0 -1 -2，奇判断的时候，当输入-1时，也会返回偶数。

### 建议10：边界、边界、还是边界

数字越界是检验条件失效，边界测试；检验条件if(order>0 && order+cur<=LIMIT)，输入的数大于0，加上cur的值之后溢出为负值，小于LIMIT，所以满足条件，但不符合要求


### 建议11：让工具类不可实例化

工具类的方法和属性都是静态的，不需要实例即可访问。实现方式：将构造函数设置为private，并且在构造函数中抛出Error错误异常


### 建议12：推荐覆写toString方法；

原始toString方法显示不人性化

### 建议13：使用package-info类为包服务；

package-info类是专门为本包服务的，是一个特殊性主要体现在3个方面：

1. 它不能随便被创建；
2. 它服务的对象很特殊；
3. package-info类不能有实现代码；

package-info类的作用：

1. 声明友好类和包内访问常量；
2. 为在包上标注注解提供便利；
3. 提供包的整体注释说明

### 建议14：正确使用String、StringBuffer、StringBuilder；

String使用“+”进行字符串连接，之前连接之后会产生一个新的对象，所以会不断的创建新对象，优化之后与StringBuilder和StringBuffer采用同样的append方法进行连接，但是每一次字符串拼接都会调用一次toString方法，所以会很耗时。StringBuffer与StringBuilder基本相同，只是一个字符数组的在扩容而已，都是可变字符序列，不同点是：StringBuffer是线程安全的，StringBuilder是线程不安全的

### 建议15：在明确的场景下，为集合指定初始容量；

ArrayList集合底层使用数组存储，如果没有初始为ArrayList指定数组大小，默认存储数组大小长度为10，添加的元素达到数组临界值后，使用Arrays.copyOf方法进行1.5倍扩容处理。HashMap是按照倍数扩容的，Stack继承自Vector，所采用扩容规则的也是翻倍

### 建议16：推荐使用枚举定义常量；

在项目开发中，推荐使用枚举常量替代接口常量和类常量）（常量分为：类常量、接口常量、枚举常量；枚举常量优点：

1. 枚举常量更简单；
2. 枚举常量属于稳态性（不允许发生越界）；
3. 枚举具有内置方法，values方法可以获取到所有枚举值；
4. 枚举可以自定义方法

### 建议84：使用构造函数协助描述枚举项；

每个枚举项都是该枚举的一个实例。可以通过添加属性，然后通过构造函数给枚举项添加更多描述信息

### 建议87：使用valueOf前必须进行校验；

（Enum.valueOf()方法会把一个String类型的名称转变为枚举项，也就是在枚举项中查找出字面值与该参数相等的枚举项。valueOf方法先通过反射从枚举类的常量声明中查找，若找到就直接返回，若找不到就抛出IllegalArgumentException异常）。

### 建议88：用枚举实现工厂方法模式更简洁；

（工厂方法模式是“创建对象的接口，让子类决定实例化哪一个类，并使一个类的实例化延迟到其子类”。枚举实现工厂方法模式有两种方法：1、枚举非静态方法实现工厂方法模式；2、通过抽象方法生成产品；优点：1、避免错误调用的发生；2、性能好，使用便捷；3、减低类间耦合性）。

### 建议92：注意@Override不同版本的区别；

（@Override注解用于方法的覆写上，它在编译期有效，也就是Java编译器在编译时会根据该注解检查方法是否真的是覆写，如果不是就报错，拒绝编译。Java1.5版本中@Override是严格遵守覆写的定义：子类方法与父类方法必须具有相同的方法名、输入参数、输出参数（允许子类缩小）、访问权限（允许子类扩大），父类必须是一个类，不是是接口，否则不能算是覆写。而在Java1.6就开放了很多，实现接口的方法也可以加上@Override注解了。如果是Java1.6版本移植到Java1.5版本中时，需要删除接口实现方法上的@Override注解）。

### 建议96：不同的场景使用不同的泛型通配符；

（Java泛型支持通配符（Wildcard），可以单独使用一个“?”表示任意类，也可以使用extends关键字表示某一个类（接口）的子类型，还可以使用super关键字表示某一个类（接口）的父类型。1、泛型结构只参与“读”操作则限定上界（extends关键字）；2、泛型结构只参与“写”操作则限定下界（使用super关键字）；3、如果一个泛型结构既用作“读”操作也用作“写”操作则使用确定的泛型类型即可，如List<E>）。

### 建议109：不需要太多关注反射效率；

（反射效率低是个真命题，但因为这一点而不使用它就是个假命题）（反射效率相对于正常的代码执行确实低很多（经测试，相差15倍左右），但是它是一个非常有效的运行期工具类）。

### 建议110：提倡异常封装；

异常封装有三方面的优点：

1. 提高系统的友好性；
2. 提高系统的可维护性；
3. 解决Java异常机制本身的缺陷）；

### 建议112：受检异常尽可能转化为非受检异常；

（受检异常威胁到系统的安全性、稳定性、可靠性、正确性时、不能转为非受检异常）（受检异常（Checked Exception），非受检异常（Unchecked Exception），受检异常时正常逻辑的一种补偿处理手段，特别是对可靠性要求比较高的系统来说，在某些条件下必须抛出受检异常以便由程序进行补偿处理，也就是说受检异常有合理的存在理由。但是受检异常有不足的地方：1、受检异常使接口声明脆弱；2、受检异常是代码的可读性降低，一个方法增加了受检异常，则必须有一个调用者对异常进行处理。受检异常需要try..catch处理；3、受检异常增加了开发工作量。避免以上受检异常缺点办法：将受检异常转化为非受检异常）。

### 建议113：不要在finally块中处理返回值；

在finally块中加入了return语句会导致以下两个问题：
1、覆盖了try代码块中的return返回值；
2、屏蔽异常，即使throw出去了异常，异常线程会登记异常，但是当执行器执行finally代码块时，则会重新为方法赋值，也就是告诉调用者“该方法执行正确”，没有发生异常，于是乎，异常神奇的消失了

### 建议124：异步运算考虑使用Callable接口；

（多线程应用的两种实现方式：一种是实现Runnable接口，另一种是继承Thread类，这两个方式都有缺点：run方法没有返回值，不能抛出异常（归根到底是Runnable接口的缺陷，Thread也是实现了Runnable接口），如果需要知道一个线程的运行结果就需要用户自行设计，线程类本身也不能提供返回值和异常。Java1.5开始引入了新的接口Callable，类似于Runnable接口，实现它就可以实现多线程任务，实现Callable接口的类，只是表明它是一个可调用的任务，并不表示它具有多线程运算能力，还是需要执行器来执行的）。

### 建议125：优先选择线程池；

（Java1.5以前，实现多线程比较麻烦，需要自己启动线程，并关注同步资源，防止出现线程死锁等问题，Java1.5以后引入了并行计算框架，大大简化了多线程开发。线程有五个状态：新建状态（New）、可运行状态（Runnable，也叫作运行状态）、阻塞状态（Blocked）、等待状态（Waiting）、结束状态（Terminated），线程的状态只能由新建转变为运行态后才可能被阻塞或等待，最后终结，不可能产生本末倒置的情况，比如想把结束状态变为新建状态，则会出现异常。线程运行时间分为三个部分：T1为线程启动时间；T2为线程体运行时间；T3为线程销毁时间。每次创建线程都会经过这三个时间会大大增加系统的响应时间。T2是无法避免的，只能通过优化代码来降低运行时间。T1和T3都可以通过线程池（Thread Pool）来缩短时间。线程池的实现涉及一下三个名词：1、工作线程（Worker），线程池中的线程只有两个状态：可运行状态和等待状态；2、任务接口（Task），每个任务必须实现的接口，以供工作线程调度器调度，它主要规定了任务的入口、任务执行完的场景处理、任务的执行状态等。这里的两种类型的任务：具有返回值（或异常）的Callable接口任务和无返回值并兼容旧版本的Runnable接口任务；3、任务队列（Work Queue），也叫作工作队列，用于存放等待处理的任务，一般是BlockingQueue的实现类，用来实现任务的排队处理。线程池的创建过程：创建一个阻塞队列以容纳任务，在第一次执行任务时闯将足够多的线程（不超过许可线程数），并处理任务，之后每个工作线程自行从任务队列中获得任务，直到任务队列中任务数量为0为止，此时，线程将处于等待状态，一旦有任务加入到队列中，即唤醒工作线程进行处理，实现线程的可复用性）。

### 建议126：适时选择不同的线程池来实现；

（Java的线程池实现从根本上来说只有两个：ThreadPoolExecutor类和ScheduledThreadPoolExecutor类，还是父子关系。为了简化并行计算，Java还提供了一个Executors的静态类，它可以直接生成多种不同的线程池执行器，比如单线程执行器、带缓冲功能的执行器等，归根结底还是以上两个类的封装类）。

### 建议133：若非必要，不要克隆对象；

### 建议138：性能是个大“咕咚”；

1. 没有慢的系统，只有不满足义务的系统；
2. 没有慢的系统，只有架构不良的系统；
3. 没有慢的系统，只有懒惰的技术人员；
4. 没有慢的系统，只有不愿意投入的系统）。

### 建议140：推荐使用Guava扩展工具包；

（Guava（石榴）是Google发布的，其中包含了collections、caching、primitives support、concurrency libraries、common annotations、I/O等）。

### 建议142：推荐使用Joda日期时间扩展包；

（Joda可以很好地与现有的日期类保持兼容，在需要复杂的日期计算时使用Joda。日期工具类也可以选择date4j）。

### 建议143：多使用工具类；


### 建议144：提倡良好的代码风格；

（良好的编码风格包括：1、整洁；2、统一；3、流行；4、便捷，推荐使用Checkstyle检测代码是否遵循规范）。

### 建议146：让注释正确、清晰、简洁；

（注释不是美化剂，而是催化剂，或为优秀加分，或为拙略减分）。

### 建议147：让接口的职责保持单一；

（接口职责一定要单一，实现类职责尽量单一）（单一职责原则（Single Responsibility Principle，简称SRP）有以下三个优点：1、类的复杂性降低；2、可读性和可维护性提高；3、降低变更风险）。

### 建议148：增强类的可替换性；

（Java的三大特征：封装、继承、多态；说说多态，一个接口可以有多种实现方式，一个父类可以有多个子类，并且可以把不同的实现或子类赋给不同的接口或父类。多态的好处非常多，其中一点就是增强了类的可替换性，但是单单一个多态特性，很难保证我们的类是完全可以替换的，幸好还有一个里氏替换原则来约束。里氏替换原则：所有引用基类的地方必须能透明地使用其子类的对象。通俗点讲，只要父类型能出现的地方子类型就可以出现，而且将父类型替换为子类型还不会产生任何错误或异常，使用者可能根本就不需要知道是父类型还是子类型。为了增强类的可替换性，在设计类时需要考虑以下三点：1、子类型必须完全实现父类型的方法；2、前置条件可以被放大；3、后置条件可以被缩小）。

### 建议149：依赖抽象而不是实现；

此处的抽象是指物体的抽象，比如出行，依赖的是抽象的运输能力，而不是具体的运输交通工具。依赖倒置原则（Dependence Inversion Principle，简称DIP）要求实现解耦，保持代码间的松耦合，提高代码的复用率。

**DIP的原始定义包含三层含义：**

1. 高层模块不应该依赖底层模块，两者都应该依赖其抽象；
2. 抽象不应该依赖细节；
3. 细节应该依赖抽象；
 
**DIP在Java语言中的表现就是：**

1. 模块间的依赖是通过抽象发生的，实现类之间不发生直接的依赖关系，其依赖关系是通过接口或抽象类产生的；
2. 接口或抽象类不依赖于实现类；
3. 实现类依赖接口或抽象类；更加精简的定义就是：面向接口编程。

**实现模块间的松耦合遵循规则：**

1. 尽量抽象；
2. 表面类型必须是抽象的；
3. 任何类都不应该从具体类派生；
4. 尽量不要覆写基类的方法；
5. 抽象不关注细节）。

### 建议150：抛弃7条不良的编码习惯；

1. 自由格式的代码；
2. 不使用抽象的代码；
3. 彰显个性的代码；
4. 死代码；
5. 冗余代码；
6. 拒绝变化的代码；
7. 自以为是的代码

### 建议151：以技术人员自律而不是工人；

1. 熟悉工具；
2. 使用IDE；
3. 坚持编码；
4. 编码前思考；
5. 坚持重构；
6. 多写文档；
7. 保持程序版本的简单性；
8. 做好备份；
9. 做单元测试；
10. 不要重复发明轮子；
11. 不要拷贝；
12. 让代码充满灵性；
13. 测试自动化；
14. 做压力测试；
15. “剽窃”不可耻；
16. 坚持向敏捷学习；
17. 重里更重面；
18. 分享；
19. 刨根问底；
20. 横向扩展
