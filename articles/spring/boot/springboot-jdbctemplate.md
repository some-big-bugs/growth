---------------------------
Spring-boot jdbctemplate	|
---------------------------
	# 简单的一个Dao层访问工具
	# 需要依赖
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-jdbc</artifactId>
			<exclusions>
				<!-- 排除掉默认的连接池 -->
				<exclusion>
					<groupId>org.apache.tomcat</groupId>
					<artifactId>tomcat-jdbc</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<!-- 使用阿里巴巴Druid连接池 -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid</artifactId>
			<version>1.0.28</version>
		</dependency>
	
	# 关于jdbctemplate的具体使用,请看 spring 系列

---------------------------
Spring-boot 配置步骤		|
---------------------------
	1,引入依赖
	2,配置 DataSource
		@Configuration
		public class DataSourceAutoConfiguration {
			@Bean
			public DataSource dataSource(){
				DruidDataSource dataSource = new DruidDataSource();
				dataSource.setUrl("jdbc:mysql://localhost:3306/community?useUnicode=true&characterEncoding=utf8&autoReconnect=true&allowMultiQueries=true");
				dataSource.setUsername("root");//用户名
				dataSource.setPassword("root");//密码
				dataSource.setInitialSize(2);
				dataSource.setMaxActive(20);
				dataSource.setMinIdle(0);
				dataSource.setMaxWait(60000);
				dataSource.setTestOnBorrow(false);
				dataSource.setTestWhileIdle(true);
				dataSource.setPoolPreparedStatements(false);
				return dataSource;
			}
		}

	3,在需要的地方注入即可
		@Autowired
		private JdbcTemplate jdbcTemplate;