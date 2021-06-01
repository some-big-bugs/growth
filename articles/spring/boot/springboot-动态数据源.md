----------------------------
几个组件					|
----------------------------
	1,配置文件,可以实现动态的配置
	2,DataSourceAspect				//AOP切面,根据事务注解来标识读/写库
	3,DynamicDataSource				//动态数据源的实现
	4,DynamicDataSourceHolder		//用于标识读/写库
	5,DataSourceConfig				//外部属性注入
	6,DataSourceAutoConfiguration	//配置到IOC

----------------------------
1,配置文件					|
----------------------------
	jdbc.datasource.masterurl=jdbc:mysql://localhost:3306/community?useUnicode=true&characterEncoding=utf8&autoReconnect=true&allowMultiQueries=true
	jdbc.datasource.masterusername=root
	jdbc.datasource.masterpassword=root

	jdbc.datasource.slaveurl[0]=jdbc:mysql://localhost:3306/community?useUnicode=true&characterEncoding=utf8&autoReconnect=true&allowMultiQueries=true
	jdbc.datasource.slaveusername[0]=root
	jdbc.datasource.slavepassword[0]=root

	jdbc.datasource.slaveurl[1]=jdbc:mysql://localhost:3306/community?useUnicode=true&characterEncoding=utf8&autoReconnect=true&allowMultiQueries=true
	jdbc.datasource.slaveusername[1]=root
	jdbc.datasource.slavepassword[1]=root

	jdbc.datasource.slaveurl[2]=jdbc:mysql://localhost:3306/community?useUnicode=true&characterEncoding=utf8&autoReconnect=true&allowMultiQueries=true
	jdbc.datasource.slaveusername[2]=root
	jdbc.datasource.slavepassword[2]=root

	jdbc.datasource.slaveurl[3]=jdbc:mysql://localhost:3306/community?useUnicode=true&characterEncoding=utf8&autoReconnect=true&allowMultiQueries=true
	jdbc.datasource.slaveusername[3]=root
	jdbc.datasource.slavepassword[3]=root

	jdbc.datasource.driverclassname=com.mysql.jdbc.Driver
	jdbc.datasource.maxactive=20
	jdbc.datasource.initialsize=10

----------------------------
2,DataSourceAspect			|
----------------------------
	package com.kevin.boot.web.datasource;
	import org.aspectj.lang.JoinPoint;
	import org.aspectj.lang.annotation.Aspect;
	import org.aspectj.lang.annotation.Before;
	import org.aspectj.lang.reflect.MethodSignature;
	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	import org.springframework.core.annotation.Order;
	import org.springframework.stereotype.Component;
	import org.springframework.transaction.annotation.Transactional;

	import java.lang.reflect.Method;
	import java.lang.reflect.ParameterizedType;
	import java.lang.reflect.Type;

	/**
	 * 动态数据源切面,必需保证该切面最先执行
	 */
	@Component
	@Aspect
	@Order(-999)        //务必保证最先执行
	public class DataSourceAspect {
		private static final Logger LOGGER = LoggerFactory.getLogger(DataSourceAspect.class);
		/**
		 * 在进入Service方法之前执行
		 * @param point
		 * @throws NoSuchMethodException
		 */
		@Before(value = "execution(* com.kevin.boot.web.service..*.*(..))")
		public void before(JoinPoint point) throws NoSuchMethodException {
			//拦截的实体对象
			Object target = point.getTarget();
			//拦截的方法名称
			String methodName = point.getSignature().getName();
			//拦截的方法参数
			Object[] args = point.getArgs();
			//拦截的方法参数类型
			Class[] parameterTypes = ((MethodSignature)point.getSignature()).getMethod().getParameterTypes();
			//获取到的目标方法
			Method method = null;
			//通过反射获得拦截的method
			method = target.getClass().getMethod(methodName, parameterTypes);
			//如果是桥则要获得实际拦截的method
			if(method.isBridge()){
				for(int i = 0; i < args.length; i++){
					//获得泛型类型
					Class genClazz = this.getSuperClassGenricType(target.getClass(),0);
					//根据实际参数类型替换parameterType中的类型
					if(args[i].getClass().isAssignableFrom(genClazz)){
						parameterTypes[i] = genClazz;
					}
				}
				//获得parameterType参数类型的方法
				method = target.getClass().getMethod(methodName, parameterTypes);
			}
			LOGGER.info("当前事务方法  " + methodName);
			Transactional transactional = method.getAnnotation(Transactional.class);
			if(transactional != null && transactional.readOnly()){
				//读库
				LOGGER.info("动态数据源 - 读库");
				DynamicDataSourceHolder.markSlave();
			}else {
				//写库
				LOGGER.info("动态数据源 - 写库");
				DynamicDataSourceHolder.markMaster();
			}
		}
		
		public Class getSuperClassGenricType(Class clazz, int index) {
			Type genType = clazz.getGenericSuperclass();        // 得到泛型父类
			if (!(genType instanceof ParameterizedType)) {
				return Object.class;
			}
			Type[] params = ((ParameterizedType) genType).getActualTypeArguments();
			if (!(params[index] instanceof Class)) {
				return Object.class;
			}
			return (Class) params[index];
		}
	}

----------------------------
3,DynamicDataSource			|
----------------------------
	package com.kevin.boot.web.datasource;

	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;
	import org.springframework.util.ReflectionUtils;

	import javax.sql.DataSource;
	import java.lang.reflect.Field;
	import java.util.ArrayList;
	import java.util.List;
	import java.util.Map;
	import java.util.concurrent.atomic.AtomicInteger;

	/**
	 * 动态数据源实现
	 */
	public class DynamicDataSource extends AbstractRoutingDataSource {
		private static final Logger LOGGER = LoggerFactory.getLogger(DynamicDataSource.class);
		/**
		 * 读库的数量
		 */
		private Integer slaveCount;
		
		// 轮询计数,初始为-1
		private AtomicInteger counter = new AtomicInteger(-1);
		
		// 记录读库的key(配置在ioc中)
		private List<Object> slaveDataSources = new ArrayList<Object>(0) ;
		
		/**
		 * 该方法的返回值,会作为一个Map<String,DataSource>中的key,去获取对应的数据源.
		 * 所以,每次请求.只要改变这个key,就可以改变当此次请求,在持久层会话中注入的数据源,实现自动的读写分离
		 */
		@Override
		protected Object determineCurrentLookupKey() {
			/**
			 * 如果是主库,或者是从库数量为0
			 */
			Object key = null;
			if (DynamicDataSourceHolder.isMaster() || this.slaveCount == 0) {
				key = DynamicDataSourceHolder.getDataSourceKey();
				return key;
			}
			key = this.getSlaveKey();
			LOGGER.info("Datasource key={}",key);
			return key;
			
		}
		/**
		 * 父类实现了InitializingBean,Spring会在Bean初始化完成后回调 afterPropertlesSet方法
		 * 通过该方法,获取到所有的读库key,存储到集合
		 */
		@SuppressWarnings("unchecked")
		@Override
		public void afterPropertiesSet() {
			super.afterPropertiesSet();
			// 由于父类的resolvedDataSources属性是私有的子类获取不到，需要使用反射获取
			Field field = ReflectionUtils.findField(AbstractRoutingDataSource.class, "resolvedDataSources");
			field.setAccessible(true); // 设置可访问
			try {
				//传递当前对象,反射执行该字段的get方法,获取到所有的数据源
				Map<Object, DataSource> resolvedDataSources = (Map<Object, DataSource>) field.get(this);
				/**
				 * 读库的数据量等于数据源总数减去写库的数量,写库数量为 1
				 */
				this.slaveCount = resolvedDataSources.size() - 1;
				for (Map.Entry<Object, DataSource> entry : resolvedDataSources.entrySet()) {
					//遍历所有的key
					if (DynamicDataSourceHolder.MASTER.equals(entry.getKey())) {
						//如果是写库,跳过之
						continue;
					}
					//添加写库的key到集合
					slaveDataSources.add(entry.getKey());
				}
			} catch (Exception e) {
				e.printStackTrace();
				LOGGER.error("afterPropertiesSet error! ", e);
			}
		}
		
		/**
		 * 轮询算法实现
		 */
		public Object getSlaveKey() {
			Integer index = counter.incrementAndGet() % slaveCount;
			if (counter.get() > 9999) { 	// 以免超出Integer范围
				counter.set(-1);		 	// 还原
			}
			return slaveDataSources.get(index);
		}
	}

----------------------------
3,DynamicDataSourceHolder	|
----------------------------
	package com.kevin.boot.web.datasource;

	/**
	 * ThreadLocal记录当前线程的DB key
	 */
	public class DynamicDataSourceHolder {
		//写库标识
		public static final String MASTER = "MASTER";
		
		//读库标识
		private static final String SLAVE = "SLAVE";
		
		//使用ThreadLocal记录当前线程的数据源标识
		private static final ThreadLocal<String> holder = new ThreadLocal<String>();
		
		/**
		 * 设置数据源标识
		 */
		private static void putDataSourceKey(String key) {
			holder.set(key);
		}
		
		/**
		 * 获取数据源key
		 */
		public static String getDataSourceKey() {
			return holder.get();
		}
		
		/**
		 * 标记写库
		 */
		public static void markMaster(){
			putDataSourceKey(MASTER);
		}
		
		/**
		 * 标记读库
		 */
		public static void markSlave(){
			putDataSourceKey(SLAVE);
		}
		
		/**
		 * 是否是主库
		 */
		public static boolean isMaster() {
			return  MASTER.equals(holder.get());
		}
	}


----------------------------
4,DataSourceConfig			|
----------------------------
	package com.kevin.boot.web.config;

	import com.alibaba.druid.pool.DruidDataSource;
	import org.springframework.boot.context.properties.ConfigurationProperties;

	import javax.sql.DataSource;
	import java.util.ArrayList;
	import java.util.List;

	/**
	 * Created by KevinBlandy on 2017/2/28 14:07
	 * 数据库配置,他妈的说好的 "属性宽松的规则" 呢？
	 */
	@ConfigurationProperties(prefix = "jdbc.datasource",ignoreInvalidFields = false)
	public class DataSourceConfig {
		private String masterurl;
		private String masterusername;
		private String masterpassword;
		
		private List<String> slaveurl = new ArrayList<String>();
		private List<String> slaveusername = new ArrayList<String>();
		private List<String> slavepassword = new ArrayList<String>();
		
		private String driverclassname;
		private int maxactive;
		private int initialsize;
		
		public String getMasterurl() {
			return masterurl;
		}
		
		public void setMasterurl(String masterurl) {
			this.masterurl = masterurl;
		}
		
		public String getMasterusername() {
			return masterusername;
		}
		
		public void setMasterusername(String masterusername) {
			this.masterusername = masterusername;
		}
		
		public String getMasterpassword() {
			return masterpassword;
		}
		
		public void setMasterpassword(String masterpassword) {
			this.masterpassword = masterpassword;
		}
		
		public List<String> getSlaveurl() {
			return slaveurl;
		}
		
		public void setSlaveurl(List<String> slaveurl) {
			this.slaveurl = slaveurl;
		}
		
		public List<String> getSlaveusername() {
			return slaveusername;
		}
		
		public void setSlaveusername(List<String> slaveusername) {
			this.slaveusername = slaveusername;
		}
		
		public List<String> getSlavepassword() {
			return slavepassword;
		}
		
		public void setSlavepassword(List<String> slavepassword) {
			this.slavepassword = slavepassword;
		}
		
		public String getDriverclassname() {
			return driverclassname;
		}
		
		public void setDriverclassname(String driverclassname) {
			this.driverclassname = driverclassname;
		}
		
		public int getMaxactive() {
			return maxactive;
		}
		
		public void setMaxactive(int maxactive) {
			this.maxactive = maxactive;
		}
		
		public int getInitialsize() {
			return initialsize;
		}
		
		public void setInitialsize(int initialsize) {
			this.initialsize = initialsize;
		}
		
		/**
		 * 根据配置获取Master连接池对象
		 * @return
		 */
		public DataSource getMasterDataSource(){
			DruidDataSource masterDataSource = new DruidDataSource();
			masterDataSource.setUrl(this.getMasterurl());
			masterDataSource.setUsername(this.getMasterusername());
			masterDataSource.setPassword(this.getMasterpassword());
			masterDataSource.setMaxActive(this.getMaxactive());
			masterDataSource.setInitialSize(this.getInitialsize());
			masterDataSource.setDriverClassName(this.getDriverclassname());
			return masterDataSource;
		}
	}


----------------------------
5,DataSourceAutoConfiguration|
----------------------------
	package com.kevin.boot.web.configuration;

	import com.alibaba.druid.pool.DruidDataSource;
	import com.kevin.boot.web.config.DataSourceConfig;
	import com.kevin.boot.web.datasource.DynamicDataSource;
	import com.kevin.boot.web.utils.GeneralUtils;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.boot.context.properties.EnableConfigurationProperties;
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Configuration;

	import javax.sql.DataSource;
	import java.util.HashMap;
	import java.util.Map;

	/**
	 * Created by KevinBlandy on 2017/2/28 14:11
	 */
	@Configuration
	@EnableConfigurationProperties(DataSourceConfig.class)
	public class DataSourceAutoConfiguration {
		
		@Autowired
		private DataSourceConfig dataSourceConfig;
		
		@Bean
		public DataSource dataSource(){
			DynamicDataSource dynamicDataSource = new DynamicDataSource();
			//从配置获取一个主库
			DataSource masterDataSource = this.dataSourceConfig.getMasterDataSource();
			//默认库为主库
			dynamicDataSource.setDefaultTargetDataSource(masterDataSource);
			Map<Object,Object> dataSources = new HashMap<Object,Object>();
			if(!GeneralUtils.isEmpty(this.dataSourceConfig.getSlaveurl()) && !GeneralUtils.isEmpty(this.dataSourceConfig.getSlaveusername()) && !GeneralUtils.isEmpty(this.dataSourceConfig.getSlavepassword())){
				int length = this.dataSourceConfig.getSlaveurl().size();
				DruidDataSource druidDataSource = null;
				for (int x = 0;x < length ; x++){
					String url = this.dataSourceConfig.getSlaveurl().get(x);
					String userName = this.dataSourceConfig.getSlaveusername().get(x);
					String passWord = this.dataSourceConfig.getSlavepassword().get(x);
					if(!GeneralUtils.isEmpty(url) && !GeneralUtils.isEmpty(userName) && !GeneralUtils.isEmpty(passWord)){
						//添加slave库
						druidDataSource = new DruidDataSource();
						druidDataSource.setUrl(url);
						druidDataSource.setUsername(userName);
						druidDataSource.setPassword(passWord);
						druidDataSource.setMaxActive(this.dataSourceConfig.getMaxactive());
						druidDataSource.setInitialSize(this.dataSourceConfig.getInitialsize());
						dataSources.put("SLAVE"+x,druidDataSource);
					}
				}
			}
			//添加主库到集合,注意此处 MASTER 值
			dataSources.put("MASTER",masterDataSource);
			dynamicDataSource.setTargetDataSources(dataSources);
			return dynamicDataSource;
		}
	}
