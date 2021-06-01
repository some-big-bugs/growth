----------------------------
Druid						|
----------------------------
	1,配置文件
		# 指定数据源类型
		spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
		spring.datasource.driver-class-name=com.mysql.jdbc.Driver
		spring.datasource.url=jdbc:mysql://120.76.182.243:1124/test
		spring.datasource.username=root
		spring.datasource.password=KevinBlandy_mysql
		
		# 连接池配置,对所有dataSource属性都生效
		spring.datasource.initialSize=10
		spring.datasource.minIdle=5
		spring.datasource.maxActive=20
		spring.datasource.maxWait=60000
		spring.datasource.timeBetweenEvictionRunsMillis=60000
		spring.datasource.minEvictableIdleTimeMillis=300000
		spring.datasource.validationQuery=SELECT 1 FROM DUAL
		spring.datasource.testWhileIdle=true
		spring.datasource.testOnBorrow=false
		spring.datasource.testOnReturn=false
		spring.datasource.poolPreparedStatements=true
		spring.datasource.maxPoolPreparedStatementPerConnectionSize=20
		spring.datasource.filters=stat,wall,log4j
		spring.datasource.connectionProperties=druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000

	3,DataSourceConfiguration
		@Configuration
		public class DataSourceConfiguration {

			@Bean(initMethod = "init")
			@ConfigurationProperties(prefix = "spring.datasource")
			public DataSource druidDataSource() {
				DruidDataSource druidDataSource = new DruidDataSource();
				return druidDataSource;
			}
		}
	
	4,监控与过滤
		
		* 记得在主线程添加,@ServletComponentScan(basePackages = {"com.xx"}) 来扫描 web组件
		* 也可以使用 springboot的web组件注册bean去注册
			ServletRegistrationBean
			FilterRegistrationBean
			ServletListenerRegistrationBean

		@WebServlet(urlPatterns = "/druid/*",initParams={@WebInitParam(name="loginUsername",value="KevinBlandy"), @WebInitParam(name="loginPassword",value="F8575532")})
		public class DruidStatViewServlet extends StatViewServlet {
			private static final long serialVersionUID = -1625661402500921518L;
		}
		
		@WebFilter(filterName="druidWebStatFilter",urlPatterns="/*",
        initParams={@WebInitParam(name="exclusions",value="*.js,*.gif,*.jpg,*.bmp,*.png,*.css,*.ico,/druid/*")})
		public class DruidStatFilter extends WebStatFilter {
		}
