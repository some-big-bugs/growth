<h1> Mybatis 配置多数据源 </h1>

---

**Table of Contents**
- [前言](#前言)
- [使用步骤](#使用步骤)
  - [1. 引入库](#1-引入库)
  - [2. 配置多数据源](#2-配置多数据源)
  - [3. 配置](#3-配置)
---

# 前言

在开发一些报表项目时，很容易涉及到从多个数据源获取数据，这里介绍一下如何给 `Mybatis` 在配置多数据源.

# 使用步骤

## 1. 引入库

正常引入 mybatis 依赖即可

## 2. 配置多数据源

```yml
spring:
  datasource:
    report1:
      driver-class-name: com.mysql.jdbc.Driver
      jdbc-url: jdbc:mysql://127.0.0.1:3306/report_1
      username: root
      password: 123456
    report2:
      driver-class-name: com.mysql.jdbc.Driver
      jdbc-url: jdbc:mysql://127.0.0.1:3306/report_2
      username: root
      password: 123456
```

## 3. 配置

**配置report1**

```java
import com.github.pagehelper.PageInterceptor;
import java.util.Properties;
import javax.sql.DataSource;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@MapperScan(basePackages = {
    "com.example.demo.mapper.report1"}, sqlSessionFactoryRef = "sqlSessionFactoryReport1")
public class ReportMybatisConfiguration {

  @Bean(name = "reportDB")
  @ConfigurationProperties(prefix = "spring.datasource.report1")
  public DataSource dataSource1() {
    return DataSourceBuilder.create().build();
  }

  @Bean
  public SqlSessionFactory sqlSessionFactoryReport1() throws Exception {
    SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
    factoryBean.setDataSource(dataSource1());

    PageInterceptor pageInterceptor = new PageInterceptor();
    Properties properties = new Properties();
    properties.setProperty("helperDialect", "mysql");
    properties.setProperty("reasonable", "true");
    properties.setProperty("supportMethodsArguments", "true");
    properties.setProperty("params", "count=countSql");
    pageInterceptor.setProperties(properties);
    factoryBean.setPlugins(pageInterceptor);

    return factoryBean.getObject();

  }

  @Bean
  public SqlSessionTemplate sqlSessionTemplateReport1() throws Exception {
    return new SqlSessionTemplate(sqlSessionFactoryReport1());
  }

}
```

**配置report2**

```java

import com.github.pagehelper.PageInterceptor;
import java.util.Properties;
import javax.sql.DataSource;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@MapperScan(basePackages = {
    "com.example.demo.mapper.report2"}, sqlSessionFactoryRef = "sqlSessionFactoryReport2")
public class Report2MybatisConfiguration {

  @Bean(name = "report2DB")
  @ConfigurationProperties(prefix = "spring.datasource.report2")
  public DataSource dataSource2() {
    return DataSourceBuilder.create().build();
  }

  @Bean
  public SqlSessionFactory sqlSessionFactoryReport2() throws Exception {
    SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
    factoryBean.setDataSource(dataSource2());

    org.apache.ibatis.session.Configuration configuration = new org.apache.ibatis.session.Configuration();
    configuration.setMapUnderscoreToCamelCase(true);
    factoryBean.setConfiguration(configuration);

    PageInterceptor pageInterceptor = new PageInterceptor();
    Properties properties = new Properties();
    properties.setProperty("helperDialect", "mysql");
    properties.setProperty("reasonable", "true");
    properties.setProperty("supportMethodsArguments", "true");
    properties.setProperty("params", "count=countSql");
    pageInterceptor.setProperties(properties);
    factoryBean.setPlugins(pageInterceptor);

    return factoryBean.getObject();

  }

  @Bean
  public SqlSessionTemplate sqlSessionTemplateReport2() throws Exception {
    return new SqlSessionTemplate(sqlSessionFactoryReport2());
  }

}
```

这样我们就配置完了，在配置的过程中加入 `PageInterceptor` 分页插件，如果不需要可以删除 `PageInterceptor` 关联代码

这样配置后，在包 `com.example.demo.mapper.report1` 编写的 `mapper` 则使用 `report1` 库, 同理在 包 `com.example.demo.mapper.report1` 编写的 `mapper` 则使用 `report2` 库。

这样就完成了多数据源的配置，同理也可以配置更多的数据源


> 参考代码 
