# 实现一个 ORM 到底多简单

---
**Table of Contents**

- [实现一个 ORM 到底多简单](#实现一个-orm-到底多简单)
- [原理](#原理)
- [ORM 实现](#orm-实现)
  - [1. 通过注解来将 Java Bean 和数据库字段关联](#1-通过注解来将-java-bean-和数据库字段关联)
  - [2. 反射工具类](#2-反射工具类)
  - [3. 简单的 model 示例](#3-简单的-model-示例)
  - [4. 注解解析](#4-注解解析)
  - [5. 数据库操作](#5-数据库操作)
  - [6. 结合反射实现查询操作](#6-结合反射实现查询操作)
- [使用动态代理实现 @Query @Select 类似功能](#使用动态代理实现-query-select-类似功能)
  - [1. 动态代理](#1-动态代理)
  - [2. 注解](#2-注解)
  - [3. 表设计](#3-表设计)
  - [4. model](#4-model)
  - [5. repository](#5-repository)
  - [7. 大体流程](#7-大体流程)
  - [8. 代理使用](#8-代理使用)
  - [9. 将生成代理放入 Spring IOC 容器中](#9-将生成代理放入-spring-ioc-容器中)
  - [10. invoke方法处理](#10-invoke方法处理)


---

在这篇文章中主要用到了注解、反射、动态代理、正则、Jexl3 表达式来实现一个 ORM, 在大多数框架中，都会用到这些东西，在这里只是简单的使用一下，而不是原理分析

# 原理

在使用的 ORM 框架中，我可以像操作对象一样操作数据的存储，这是怎么实现的，我们知道数据库是认识 SQL 语句的，但并不认识`java bean` 呀！同时我们在使用ORM时，需要根据ORM框架的规定定义我们的bean，这是为什么？

这是因为 `ORM` 为我们提供了将对象操作转化为对应的 `SQL`语句，例如 `save(bean)`, 这时就需要转化成一个 `insert` 语句，`update(bean)` 这时就需要转成成对应的 update 语句

通常 insert 语句格式为 
```sql
insert into 表名 (列名) values( 值)
```
update 语句为 
```sql
update 表名 set 列名 = 值 where id = 值
```
上面的格式可以看出，如果我们能从对象中得出 `表名` `列名` `值`，我们也可以写一个简单的ORM框架


# ORM 实现

## 1. 通过注解来将 Java Bean 和数据库字段关联

上篇文章中提到了一下注解，以及自定义注解和解析注解的方法，通过使用注解，我们可以完成一个简单的ORM

要想实现对数据库的操作，我们必须知道数据表名以及表中的字段名称以及类型，正如hibernate 使用注解标识 model 与数据库的映射关系一样，这里我使用了`Java Persistence API`

**注解说明**


注解 | 作用 | 使用说明
---------|----------|---------
 @Entity | 标记这是一个实体 | 标注在类上，标明该类可以被 ORM 处理 
 @Table | 标记实体对应的表 | 标注在类上，标明该类对应的数据库标明
 @Id | 标记该字段是id | 标注在字段上，标明该字段为 id 
 @Column | 标记该字段对应的列信息 | 标记在字段上，标明对应的列信息，主要对应列名


**字段属性表**
用来存储对象中字段与数据表列的对应关系
```java
package com.zyndev.tool.fastsql.core;

import lombok.Data;

import javax.persistence.GenerationType;

/**
 * The type Db column info.
 *
 * @author 张瑀楠 zyndev@gmail.com
 * @version 0.0.1
 */
@Data
class DBColumnInfo {

    /**
     * (Optional) The primary key generation strategy
     * that the persistence provider must use to
     * generate the annotated entity primary key.
     */
    private GenerationType strategy = GenerationType.AUTO;

    private String fieldName;

    /**
     * The name of the column
     */
    private String columnName;

    /**
     * Whether the column is a unique key.
     */
    private boolean unique;

    /**
     * Whether the database column is nullable.
     */
    private boolean nullable = true;

    /**
     * Whether the column is included in SQL INSERT
     */
    private boolean insertAble = true;

    /**
     * Whether the column is included in SQL UPDATE
     */
    private boolean updatable = true;

    /**
     * The SQL fragment that is used when
     * generating the DDL for the column.
     */
    private String columnDefinition;

    /**
     * The name of the table that contains the column.
     * If absent the column is assumed to be in the primary table.
     */
    private String table;

    /**
     * (Optional) The column length. (Applies only if a
     * string-valued column is used.)
     */
    private int length =  255;

    private boolean id = false;

}
```
## 2. 反射工具类
提供一些常用的反射操作

通过反射我们可以动态的得到一个类所有的成员变量的信息，同时为这些变量取值或者赋值

```java
package com.zyndev.tool.fastsql.util;


import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;


/**
 * The type Bean reflection util.
 *
 * @author yunan.zhang zyndev@gmail.com
 * @version 0.0.3
 * @date 2017年12月26日 16点36分
 */
public class BeanReflectionUtil {

    public static Object getPrivatePropertyValue(Object obj,String propertyName)throws Exception{
        Class cls=obj.getClass();
        Field field=cls.getDeclaredField(propertyName);
        field.setAccessible(true);
        Object retvalue=field.get(obj);
        return retvalue;
    }

    /**
     * Gets static field value.
     */
    public static Object getStaticFieldValue(String className, String fieldName) throws Exception {
        Class cls = Class.forName(className);
        Field field = cls.getField(fieldName);
        return field.get(cls);
    }

    /**
     * Gets field value.
     */
    public static Object getFieldValue(Object obj, String fieldName) throws Exception {
        Class cls = obj.getClass();
        Field field = cls.getDeclaredField(fieldName);
        field.setAccessible(true);
        return field.get(obj);
    }

    /**
     * Invoke method object.
     */
    public Object invokeMethod(Object owner, String methodName, Object[] args) throws Exception {
        Class cls = owner.getClass();
        Class[] argclass = new Class[args.length];
        for (int i = 0, j = argclass.length; i < j; i++) {
            argclass[i] = args[i].getClass();
        }
        @SuppressWarnings("unchecked")
        Method method = cls.getMethod(methodName, argclass);
        return method.invoke(owner, args);
    }

    /**
     * Invoke static method object.
     */
    public Object invokeStaticMethod(String className, String methodName, Object[] args) throws Exception {
        Class cls = Class.forName(className);
        Class[] argClass = new Class[args.length];
        for (int i = 0, j = argClass.length; i < j; i++) {
            argClass[i] = args[i].getClass();
        }
        @SuppressWarnings("unchecked")
        Method method = cls.getMethod(methodName, argClass);
        return method.invoke(null, args);
    }

    /**
     * New instance object.
     */
    public static Object newInstance(String className) throws Exception {
        Class clazz = Class.forName(className);
        @SuppressWarnings("unchecked")
        java.lang.reflect.Constructor cons = clazz.getConstructor();
        return cons.newInstance();
    }

    /**
     * New instance object.
     */
    public static Object newInstance(Class clazz) throws Exception {
        @SuppressWarnings("unchecked")
        java.lang.reflect.Constructor cons = clazz.getConstructor();
        return cons.newInstance();
    }

    /**
     * Get bean declared fields field [ ].
     */
    public static Field[] getBeanDeclaredFields(String className) throws ClassNotFoundException {
        Class clazz = Class.forName(className);
        return clazz.getDeclaredFields();
    }

    /**
     * Get bean declared methods method [ ].
     */
    public static Method[] getBeanDeclaredMethods(String className) throws ClassNotFoundException {
        Class clazz = Class.forName(className);
        return clazz.getMethods();
    }

    /**
     * Copy fields.
     */
    public static void copyFields(Object source, Object target) {
        try {
            List<Field> list = new ArrayList<>();
            Field[] sourceFields = getBeanDeclaredFields(source.getClass().getName());
            Field[] targetFields = getBeanDeclaredFields(target.getClass().getName());
            Map<String, Field> map = new HashMap<>(targetFields.length);
            for (Field field : targetFields) {
                map.put(field.getName(), field);
            }
            for (Field field : sourceFields) {
                if (map.get(field.getName()) != null) {
                    list.add(field);
                }
            }
            for (Field field : list) {
                Field tg = map.get(field.getName());
                tg.setAccessible(true);
                tg.set(target, getFieldValue(source, field.getName()));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
## 3. 简单的 model 示例

```java
package com.zyndev.tool.fastsql;

import javax.persistence.*;

/**
 * @author 张瑀楠 zyndev@gmail.com
 * @date   2017/11/30 下午11:21
 */
@Data
@Entity
@Table(name = "tb_student")
public class Student {

    @Id
    @Column
    private Integer id;

    @Column
    private String name;

    @Column(updatable = true, insertable = true, nullable = false)
    private Integer age;

}
```

## 4. 注解解析

将对象的上的注解进行解析，得到对应关系
```java
package com.zyndev.tool.fastsql.core;


import com.zyndev.tool.fastsql.util.StringUtil;

import javax.persistence.Column;
import javax.persistence.Id;
import javax.persistence.Table;
import java.lang.reflect.Field;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;


/**
 * The type Annotation parser.
 * <p>
 * 注解解析工具
 *
 * @author yunan.zhang zyndev@gmail.com
 * @version 0.0.3
 * @since 2017-12-26 16:59:07
 */
public class AnnotationParser {

    /**
     * 存储类名和数据库表名的关系
     * 使用三个cache 主要为了减少反射的次数，提高效率
     */
    private final static Map<String, String> tableNameCache = new HashMap<>(30);

    /**
     * 存储类名和数据库列的关联关系
     */
    private final static Map<String, String> tableAllColumnNameCache = new HashMap<>(30);

    /**
     * 存储类名和对应的数据库全列名的关系
     */
    private final static Map<String, List<DBColumnInfo>> tableAllDBColumnCache = new HashMap<>(30);

    /**
     * Gets table name.
     * 获得表名
     * 判断是否有@Table注解，如果有得到注解的name,如果name为空，则使用类名做为表名
     * 如果没有@Table返回null
     *
     * @param <E>    the type parameter
     * @param entity the entity
     * @return the table name
     */
    public static <E> String getTableName(E entity) {
        String tableName = tableNameCache.get(entity.getClass().getName());
        if (tableName == null) {
            Table table = entity.getClass().getAnnotation(Table.class);
            if (table != null && StringUtil.isNotBlank(table.name())) {
                tableName = table.name();
            } else {
                tableName = entity.getClass().getSimpleName();
            }
            tableNameCache.put(entity.getClass().getName(), tableName);
        }
        return tableName;
    }

    /**
     * Gets all db column info.
     */
    public static <E> List<DBColumnInfo> getAllDBColumnInfo(E entity) {
        List<DBColumnInfo> dbColumnInfoList = tableAllDBColumnCache.get(entity.getClass().getName());
        if (dbColumnInfoList == null) {
            dbColumnInfoList = new ArrayList<>();
            DBColumnInfo dbColumnInfo;
            Field[] fields = entity.getClass().getDeclaredFields();
            for (Field field : fields) {
                Column column = field.getAnnotation(Column.class);
                if (column != null) {
                    dbColumnInfo = new DBColumnInfo();
                    if (StringUtil.isBlank(column.name())) {
                        dbColumnInfo.setColumnName(field.getName());
                    } else {
                        dbColumnInfo.setColumnName(column.name());
                    }
                    if (null != field.getAnnotation(Id.class)) {
                        dbColumnInfo.setId(true);
                    }
                    dbColumnInfo.setFieldName(field.getName());
                    dbColumnInfoList.add(dbColumnInfo);
                }
            }
            tableAllDBColumnCache.put(entity.getClass().getName(), dbColumnInfoList);
        }
        return dbColumnInfoList;
    }

    /**
     * 返回表字段的所有字段  column1,column2,column3
     *
     * @param <E>    the type parameter
     * @param entity the entity
     * @return string
     */
    public static <E> String getTableAllColumn(E entity) {

        String allColumn = tableAllColumnNameCache.get(entity.getClass().getName());
        if (allColumn == null) {
            List<DBColumnInfo> dbColumnInfoList = getAllDBColumnInfo(entity);
            StringBuilder allColumnsInfo = new StringBuilder();
            int i = 1;
            for (DBColumnInfo dbColumnInfo : dbColumnInfoList) {
                allColumnsInfo.append(dbColumnInfo.getColumnName());
                if (i != dbColumnInfoList.size()) {
                    allColumnsInfo.append(",");
                }
                i++;
            }
            allColumn = allColumnsInfo.toString();
            tableAllColumnNameCache.put(entity.getClass().getName(), allColumn);
        }
        return allColumn;

    }
}
```
## 5. 数据库操作

数据库交互使用spring 提供的 JdbcTemplate，这里没有在自己写一套 DBUtil

## 6. 结合反射实现查询操作

> 保存一个entity

保存操作相对简单，这里主要是将 entity 转换为 insert 语句

```java
/**
 * Save int.
 * @param entity the entity
 * @return the int
 */
@Override
public int save(Object entity) {
    try {
        String tableName = AnnotationParser.getTableName(entity);
        StringBuilder property = new StringBuilder();
        StringBuilder value = new StringBuilder();
        List<Object> propertyValue = new ArrayList<>();
        List<DBColumnInfo> dbColumnInfoList = AnnotationParser.getAllDBColumnInfo(entity);

        for (DBColumnInfo dbColumnInfo : dbColumnInfoList) {
            if (dbColumnInfo.isId() || !dbColumnInfo.isInsertAble()) {
                continue;
            }
            // 不为null
            Object o = BeanReflectionUtil.getFieldValue(entity, dbColumnInfo.getFieldName());
            if (o != null) {
                property.append(",").append(dbColumnInfo.getColumnName());
                value.append(",").append("?");
                propertyValue.add(o);
            }
        }
        String sql = "insert into " + tableName + "(" + property.toString().substring(1) + ") values(" + value.toString().substring(1) + ")";
        return this.getJdbcTemplate().update(sql, propertyValue.toArray());
    } catch (Exception e) {
        e.printStackTrace();
    }
    return 0;
}

```

> 更新操作

更新操作相对于 保存来说，多了一步确定`where` 语句操作

```java
/**
 * Update int.
 *
 * @param entity     the entity
 * @param ignoreNull the ignore null
 * @param columns    the columns
 * @return the int
 */
@Override
public int update(Object entity, boolean ignoreNull, String... columns) {
    try {
        String tableName = AnnotationParser.getTableName(entity);
        StringBuilder property = new StringBuilder();
        StringBuilder where = new StringBuilder();
        List<Object> propertyValue = new ArrayList<>();
        List<Object> wherePropertyValue = new ArrayList<>();
        List<DBColumnInfo> dbColumnInfos = AnnotationParser.getAllDBColumnInfo(entity);
        for (DBColumnInfo dbColumnInfo : dbColumnInfos) {

            Object o = BeanReflectionUtil.getFieldValue(entity, dbColumnInfo.getFieldName());
            if (dbColumnInfo.isId()) {
                where.append(" and ").append(dbColumnInfo.getColumnName()).append(" = ? ");
                wherePropertyValue.add(o);
            } else if (ignoreNull || o != null) {
                property.append(",").append(dbColumnInfo.getColumnName()).append("=?");
                propertyValue.add(o);
            }
        }

        if (wherePropertyValue.isEmpty()) {
            throw new IllegalArgumentException("更新表 [" + tableName + "] 无法找到id, 请求数据：" + entity);
        }

        String sql = "update " + tableName + " set " + property.toString().substring(1) + " where " + where.toString().substring(5);
        propertyValue.addAll(wherePropertyValue);
        return this.getJdbcTemplate().update(sql, propertyValue.toArray());
    } catch (Exception e) {
        e.printStackTrace();
    }
    return 0;
}
```

> 删除操作

相对 update 简单一点

```
/**
 * Delete int.
 * <p>根据id 删除对应的数据</p>
 *
 * @param entity the entity
 * @return the int
 */
@Override
public int delete(Object entity) {
    try {
        String tableName = AnnotationParser.getTableName(entity);
        StringBuilder where = new StringBuilder(" 1=1 ");
        List<Object> whereValue = new ArrayList<>(5);
        List<DBColumnInfo> dbColumnInfos = AnnotationParser.getAllDBColumnInfo(entity);
        for (DBColumnInfo dbColumnInfo : dbColumnInfos) {
            if (dbColumnInfo.isId()) {
                Object o = BeanReflectionUtil.getFieldValue(entity, dbColumnInfo.getFieldName());
                if (null != o) {
                    whereValue.add(o);
                }
                where.append(" and `").append(dbColumnInfo.getColumnName()).append("` = ? ");
            }
        }

        if (whereValue.size() == 0) {
            throw new IllegalStateException("delete " + tableName + " id 无对应值，不能删除");
        }
        String sql = "delete from  " + tableName + " where " + where.toString();
        return this.getJdbcTemplate().update(sql, whereValue);
    } catch (Exception e) {
        e.printStackTrace();
    }
    return 0;
}
```

> 通过上面的示例，就可以简单的实现一个 ORM, 为了更好的使用，我们还需要提供自己写 SQL 的方案

# 使用动态代理实现 @Query @Select 类似功能

## 1. 动态代理

这里直接使用基于 JDK 动态代理实现

## 2. 注解

在 `Java Persistence API` 中没有我们需要的 @Query 和 @Param 这里我们自定义一下这两个注解，同时为了让 Query 支持返回主键，在定义一个 **`ReturnGeneratedKey`** 注解

**Query.java**
```java
package com.zyndev.tool.fastsql.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 查询操作 sql
 *
 * @author 张瑀楠 zyndev@gmail.com
 * @version 0.0.1
 * @since  2017/12/22 17:26
 */
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface Query {

    /**
     * sql 语句
     */
    String value();
}
```

**Param.java**

```java
package com.zyndev.tool.fastsql.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 *
 * @author 张瑀楠 zyndev@gmail.com
 * @version 0.0.1
 * @since  2017/12/22 17:29
 */
@Target( { ElementType.PARAMETER })
@Retention(RetentionPolicy.RUNTIME)
public @interface Param {

    /**
     * 指出这个值是 SQL 语句中哪个参数的值，使用命名参数
     */
    String value();
}

```

**ReturnGeneratedKey.java**

```java
package com.zyndev.tool.fastsql.annotation;


import java.lang.annotation.*;


/**
 * 返回主键
 *
 * @author 张瑀楠 zyndev@gmail.com
 * @version 0.0.3
 *
 */
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ReturnGeneratedKey {
}
```

## 3. 表设计

```sql
CREATE TABLE `tb_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `uid` varchar(40) DEFAULT NULL,
  `account_name` varchar(40) DEFAULT NULL,
  `nick_name` varchar(23) DEFAULT NULL,
  `password` varchar(30) DEFAULT NULL,
  `phone` varchar(16) DEFAULT NULL,
  `register_time` timestamp NULL DEFAULT NULL,
  `update_time` timestamp NULL DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8
```

## 4. model

这里使用一个 User.java 作为例子：

**User.java**

```java
package com.zyndev.tool.fastsql.repository;

import lombok.*;

import java.io.Serializable;
import java.util.Date;

/**
 * @author 张瑀楠 zyndev@gmail.com
 * @version 1.0
 * @date 2017-12-27 15:04:13
 */
@Data
public class User implements Serializable {

    private static final long serialVersionUID = 1L;

    private Integer id;

    private String uid;

    private String accountName;

    private String nickName;

    private String password;

    private String phone;

    private Date registerTime;

    private Date updateTime;

}
```

## 5. repository

```java
package com.zyndev.tool.fastsql.repository;

import com.zyndev.tool.fastsql.annotation.Param;
import com.zyndev.tool.fastsql.annotation.Query;
import com.zyndev.tool.fastsql.annotation.ReturnGeneratedKey;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Map;

/**
 * 这里应该有描述
 *
 * @version 1.0
 * @author 张瑀楠 zyndev@gmail.com
 * @date 2017 /12/22 18:13
 */
@Repository
public interface UserRepository {

    @Query("select count(*) from tb_user")
    public Integer getCount();

    @Query("delete from tb_user where id = ?1")
    public Boolean deleteById(int id);

    @Query("select count(*) from tb_user where password = ?1 ")
    public int getCountByPassword(@Param("password") String password);

    @Query("select uid from tb_user where password = ?1 ")
    public String getUidByPassword(@Param("password") String password);

    @Query("select * from tb_user where id = :id ")
    public User getUserById(@Param("id") Integer id);

    @Query("select * " +
            " from tb_user " +
            " where account_name = :accountName ")
    public List<User> getUserByAccountName(@Param("accountName") String accountName);

    @Query("insert into tb_user(id, account_name, password, uid, nick_name, register_time, update_time) " +
            "values(:id, :user.accountName, :user.password, :user.uid, :user.nickName, :user.registerTime, :user.updateTime )")
    public int saveUser(@Param("id") Integer id, @Param("user") User user);

    @ReturnGeneratedKey
    @Query("insert into tb_user(account_name, password, uid, nick_name, register_time, update_time) " +
            "values(:user.accountName, :user.password, :user.uid, :user.nickName, :user.registerTime, :user.updateTime )")
    public int saveUser(@Param("user") User user);
}
```

## 7. 大体流程

在上面的我们已经完成一些准备工作，包括：

1. 注解的定义
1. 表的设计
1. model 的设计
1. Repository 的设计

接下来，我们看看如何将这些整合在一起

大致流程：
1. 为 Repository 生成代理
1. 将生成代理放入 Spring IOC 容器中
1. 当代理的方法被调用时，得到方法的 `@Query` , `@Param`,`@ReturnGeneratedKey` 注解，并取得方法的返回值
1. 重写 Query的sql，并执行，根据方法的返回类型，封装SQL返回结果集

## 8. 代理使用

**FacadeProxy.java**

为 Repository 生成代理，当代理方法执行时，回调 invoke 方法，invoke 中逻辑写到**StatementParser.java**中，防止类功能过大

```java
package com.zyndev.tool.fastsql.core;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/**
 * 生成代理
 *
 * @author 张瑀楠 zyndev@gmail.com
 * @version 0.0.1
 * @since  2017 /12/23 上午12:40
 */
@SuppressWarnings("unchecked")
public class FacadeProxy implements InvocationHandler {

    private final static Log logger = LogFactory.getLog(FacadeProxy.class);

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        return StatementParser.invoke(proxy, method, args);
    }

    /**
     * New mapper proxy t.
     */
    protected static <T> T newMapperProxy(Class<T> mapperInterface) {
        logger.info(" 生成代理：" + mapperInterface.getName());
        ClassLoader classLoader = mapperInterface.getClassLoader();
        Class<?>[] interfaces = new Class[]{mapperInterface};
        FacadeProxy proxy = new FacadeProxy();
        return (T) Proxy.newProxyInstance(classLoader, interfaces, proxy);
    }
}
```

## 9. 将生成代理放入 Spring IOC 容器中

在这里使用了 `BeanFactoryAware`,关于这部分内容会单独写一篇，这里不在详细说明

```java
package com.zyndev.tool.fastsql.core;

import com.zyndev.tool.fastsql.util.ClassScanner;
import com.zyndev.tool.fastsql.util.StringUtil;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.BeanFactory;
import org.springframework.beans.factory.BeanFactoryAware;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.context.annotation.ImportBeanDefinitionRegistrar;
import org.springframework.core.type.AnnotationMetadata;
import org.springframework.stereotype.Repository;

import java.io.IOException;
import java.util.Set;

/**
 * The type Fast sql repository registrar.
 *
 * @author 张瑀楠 zyndev@gmail.com
 * @version 1.0
 * @date 2018 /2/23 12:26
 */
public class FastSqlRepositoryRegistrar implements ImportBeanDefinitionRegistrar, BeanFactoryAware {

    private ConfigurableListableBeanFactory beanFactory;

    @Override
    public void registerBeanDefinitions(AnnotationMetadata annotationMetadata, BeanDefinitionRegistry beanDefinitionRegistry) {
        System.out.println("FastSqlRepositoryRegistrar registerBeanDefinitions ");
        String basePackage = "com.sparrow";
        ClassScanner classScanner = new ClassScanner();
        Set<Class<?>> classSet = null;
        try {
            classSet = classScanner.getPackageAllClasses(basePackage, true);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
        for (Class clazz : classSet) {
            if (clazz.getAnnotation(Repository.class) != null) {
                beanFactory.registerSingleton(StringUtil.firstCharToLowerCase(clazz.getSimpleName()), FacadeProxy.newMapperProxy(clazz));
            }
        }
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = (ConfigurableListableBeanFactory) beanFactory;
    }
}

```

## 10. invoke方法处理

在前面生成动态的代理的时候，可以看到，所有的**invoke**逻辑由**StatementParser.java**处理，这也是一个重量级的方法

> invoke执行流程说明：

**invoke(Object proxy, Method method, Object[] args)**
1. 得到方法的返回类型
1. 得到方法的**@Query**注解，取得需要执行的`sql`语句，无法取到sql则抛异常
1. 获得方法的参数，并将参数顺序对应为 ?1->arg0 ?2->arg1 ...
1. 获得方法的参数和参数上`@Param`注解，并将参数与对应的Param的名称关联：param1->arg0 password->arg1 
1. 判断sql是select还是其他，使用正则 `(?i)select([\\s\\S]*?)`
1. 重写sql
1. 如果不是 select 语句，判断是否是 `@ReturnGeneratedKey` 注解
1. 如果无 `@ReturnGeneratedKey` 则直接执行语句并返回对应的结果
1. 如有有 `@ReturnGeneratedKey` 并且是 insert 语句则返回生成的主键
1. 如果是 select 语句，则执行select 语句，并根据方法的返回类型封装结果集


> 关于重写sql

```java
@Query("insert into tb_user(id, 
account_name, 
password, 
uid, 
nick_name, 
register_time, 
update_time) values(
    :id, 
    :user.accountName, 
    :user.password, 
    :user.uid, 
    :user.nickName, 
    :user.registerTime, 
    :user.updateTime )")
    public int saveUser(@Param("id") Integer id, @Param("user") User user); 
```

首先获取sql

```sql
insert into tb_user(id, account_name, password, uid, nick_name, register_time, update_time)
values(:id, :user.accountName, :user.password, :user.uid, :user.nickName, :user.registerTime, :user.updateTime )
```

可以看出这并不是标准的 sql 也不是 jdbc 可以识别的sql，这里我们使用正则**\?\d+(\.[A-Za-z]+)?|:[A-Za-z0-9]+(\.[A-Za-z]+)?**

来提取出 ?1 ?2 :1 :2 :id :user.accountName 的特殊标志，并将其替换为 ？

替换过程中没替换一个 ？ 则记录对应的 ？所代表的数值 

替换后的SQL为
```sql
insert into tb_user(id, account_name, password, uid, nick_name, register_time, update_time)
values(?, ?, ?, ?, ?, ?, ? )
```
这样的sql就可以被 jdbc 处理了
同时参数允许为：
```
:id, :user.accountName, :user.password, :user.uid, :user.nickName, :user.registerTime, :user.updateTime
```
这里的 id 可以从参数中 id 直接获取，
`:user.accountName` 则需要从参数 `:user` 即 user 中通过反射获取，这样 SQL 的重写就完成了

> 返回结果集封装可以通过反射，可以直接看下面代码

**StatementParser.java**

```java
package com.zyndev.tool.fastsql.core;

import com.sun.org.apache.bcel.internal.generic.IF_ACMPEQ;
import com.zyndev.tool.fastsql.annotation.Param;
import com.zyndev.tool.fastsql.annotation.Query;
import com.zyndev.tool.fastsql.annotation.ReturnGeneratedKey;
import com.zyndev.tool.fastsql.convert.BeanConvert;
import com.zyndev.tool.fastsql.convert.ListConvert;
import com.zyndev.tool.fastsql.convert.SetConvert;
import com.zyndev.tool.fastsql.util.BeanReflectionUtil;
import com.zyndev.tool.fastsql.util.StringUtil;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.springframework.jdbc.core.*;
import org.springframework.jdbc.support.GeneratedKeyHolder;
import org.springframework.jdbc.support.KeyHolder;
import org.springframework.jdbc.support.rowset.SqlRowSet;
import sun.reflect.generics.reflectiveObjects.NotImplementedException;

import java.lang.reflect.Method;
import java.lang.reflect.Parameter;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * sql 语句解析
 * <p>
 * 暂时只能处理 select count(*) from tb_user 类似语句
 *
 * @author 张瑀楠 zyndev@gmail.com
 * @version 0.0.1
 * @since 2017 /12/23 下午12:11
 */
class StatementParser {

    private final static Log logger = LogFactory.getLog(StatementParser.class);

    private static PreparedStatementCreator getPreparedStatementCreator(final String sql, final Object[] args, final boolean returnKeys) {
        PreparedStatementCreator creator = new PreparedStatementCreator() {

            @Override
            public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
                PreparedStatement ps = con.prepareStatement(sql);
                if (returnKeys) {
                    ps = con.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS);
                } else {
                    ps = con.prepareStatement(sql);
                }

                if (args != null) {
                    for (int i = 0; i < args.length; i++) {
                        Object arg = args[i];
                        if (arg instanceof SqlParameterValue) {
                            SqlParameterValue paramValue = (SqlParameterValue) arg;
                            StatementCreatorUtils.setParameterValue(ps, i + 1, paramValue,
                                    paramValue.getValue());
                        } else {
                            StatementCreatorUtils.setParameterValue(ps, i + 1,
                                    SqlTypeValue.TYPE_UNKNOWN, arg);
                        }
                    }
                }
                return ps;
            }
        };
        return creator;
    }

    /**
     * 此处对 Repository 中方法进行解析，解析成对应的sql 和 参数
     * <p>
     * sql 来自于 @Query 注解的 value
     * 参数 来自方法的参数
     * <p>
     * 注意根据返回值的不同封装结果集
     *
     * @param proxy  执行对象
     * @param method 执行方法
     * @param args   参数
     * @return object
     */
    static Object invoke(Object proxy, Method method, Object[] args) throws Exception {

        JdbcTemplate jdbcTemplate = DataSourceHolder.getInstance().getJdbcTemplate();

        boolean logDebug = logger.isDebugEnabled();

        String methodReturnType = method.getReturnType().getName();
        Query query = method.getAnnotation(Query.class);

        if (null == query || StringUtil.isBlank(query.value())) {
            logger.error(method.toGenericString() + " 无 query 注解或 SQL 为空");
            throw new IllegalStateException(method.toGenericString() + " 无 query 注解或 SQL 为空");
        }

        String originSql = query.value().trim();

        System.out.println("sql:" + query.value());
        Map<String, Object> namedParamMap = new HashMap<>();
        Parameter[] parameters = method.getParameters();
        if (args != null && args.length > 0) {
            for (int i = 0; i < args.length; ++i) {
                Param param = parameters[i].getAnnotation(Param.class);
                if (null != param) {
                    namedParamMap.put(param.value(), args[i]);
                }
                namedParamMap.put("?" + (i + 1), args[i]);
            }
        }

        if (logDebug) {
            logger.debug("执行 sql: " + originSql);
        }

        // 判断 sql 类型, 判断是否为 select 开头语句
        boolean isQuery = originSql.trim().matches("(?i)select([\\s\\S]*?)");
        Object[] params = null;
        // rewrite sql
        if (null != args && args.length > 0) {
            List<String> results = StringUtil.matches(originSql, "\\?\\d+(\\.[A-Za-z]+)?|:[A-Za-z0-9]+(\\.[A-Za-z]+)?");
            if (results.isEmpty()) {
                params = args;
            } else {
                params = new Object[results.size()];
                for (int i = 0; i < results.size(); ++i) {
                    if (results.get(i).charAt(0) == ':') {
                        originSql = originSql.replaceFirst(results.get(i), "?");
                        // 判断是否是 param.param 的格式
                        if (!results.get(i).contains(".")) {
                            params[i] = namedParamMap.get(results.get(i).substring(1));
                        } else {
                            String[] paramArgs = results.get(i).split("\\.");
                            Object param = namedParamMap.get(paramArgs[0].substring(1));
                            params[i] = BeanReflectionUtil.getFieldValue(param, paramArgs[1]);
                        }
                        continue;
                    }
                    int paramIndex = Integer.parseInt(results.get(i).substring(1));
                    originSql = originSql.replaceFirst("\\?" + paramIndex, "?");
                    params[i] = namedParamMap.get(results.get(i));
                }
            }
        }


        System.out.println("execute sql:" + originSql);
        System.out.print("params : ");
        if (null != params) {
            for (Object o : params) {
                System.out.print(o + ",\t");
            }
        }
        System.out.println("\n");


        /**
         * 如果返回值是基本类型或者其包装类
         */
        System.out.println(methodReturnType);
        if (isQuery) {
            // 查询方法
            if ("java.lang.Integer".equals(methodReturnType) || "int".equals(methodReturnType)) {
                return jdbcTemplate.queryForObject(originSql, params, Integer.class);
            } else if ("java.lang.String".equals(methodReturnType)) {
                return jdbcTemplate.queryForObject(originSql, params, String.class);
            } else if ("java.util.List".equals(methodReturnType) || "java.util.Set".equals(methodReturnType)) {
                String typeName = null;
                Type returnType = method.getGenericReturnType();
                if (returnType instanceof ParameterizedType) {
                    Type[] types = ((ParameterizedType) returnType).getActualTypeArguments();
                    if (null == types || types.length > 1) {
                        throw new IllegalArgumentException("当返回值为 list 时，必须标明具体类型，且只有一个");
                    }
                    typeName = types[0].getTypeName();
                }
                Object obj = BeanReflectionUtil.newInstance(typeName);
                SqlRowSet rowSet = jdbcTemplate.queryForRowSet(originSql, params);
                if ("java.util.List".equals(methodReturnType)) {
                    return ListConvert.convert(rowSet, obj);
                }
                return SetConvert.convert(rowSet, obj);
            } else if ("java.util.Map".equals(methodReturnType)) {
                throw new NotImplementedException();
            } else {
                SqlRowSet rowSet = jdbcTemplate.queryForRowSet(originSql, params);
                Object obj = BeanReflectionUtil.newInstance(methodReturnType);
                return BeanConvert.convert(rowSet, obj);
            }
        } else {
            // 非查询方法
            // 判断是否是insert 语句
            ReturnGeneratedKey returnGeneratedKeyAnnotation = method.getAnnotation(ReturnGeneratedKey.class);
            if (returnGeneratedKeyAnnotation == null) {
                int retVal = jdbcTemplate.update(originSql, params);
                if ("java.lang.Integer".equals(methodReturnType) || "int".equals(methodReturnType)) {
                    return retVal;
                } else if ("java.lang.Boolean".equals(methodReturnType)) {
                    return retVal > 0;
                }
            } else {
                // 判断是否是 insert 语句
                boolean isInsertSql = originSql.trim().matches("(?i)insert([\\s\\S]*?)");
                if (isInsertSql) {
                    KeyHolder keyHolder = new GeneratedKeyHolder();
                    PreparedStatementCreator preparedStatementCreator = getPreparedStatementCreator(originSql, params, true);
                    jdbcTemplate.update(preparedStatementCreator, keyHolder);
                    if ("java.lang.Integer".equals(methodReturnType) || "int".equals(methodReturnType)) {
                        return keyHolder.getKey().intValue();
                    } else if ("java.lang.Long".equals(methodReturnType) || "long".equals(methodReturnType)) {
                        return keyHolder.getKey().longValue();
                    }
                    logger.error(method.toGenericString() + " 返回主键id应该为 int 或者 long 类型 ");
                    throw new IllegalArgumentException(method.toGenericString() + " 返回主键id应该为 int 或者 long 类型 ");
                } else {
                    logger.error(method.toGenericString() + " 非 insert 语句 无法返回 GeneratedKey： sql语句为：" + originSql);
                    throw new IllegalStateException(method.toGenericString() + " 非 insert 语句 无法返回 GeneratedKey： sql语句为：" + originSql);
                }
            }
        }
        return null;
    }
}

```

由此一个简单 ORM 就实现了，其实实现 ORM 并不难，难的是细心处理各种可能的 Bug