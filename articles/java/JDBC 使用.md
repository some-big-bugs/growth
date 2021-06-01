<h1> JDBC (Java DataBase Connectivity) </h1>

---

**Table of Contents**

- [简介](#简介)
- [使用 `JDBC` 完成一个简单的数据库操作](#使用-jdbc-完成一个简单的数据库操作)
- [关于事务](#关于事务)
  - [概念](#概念)
  - [特性](#特性)
  - [提交和回滚](#提交和回滚)
  - [保存点](#保存点)
- [API 参考](#api-参考)
  - [Connection](#connection)
  - [Statement](#statement)
  - [ResultSet](#resultset)
  - [PreparedStatement](#preparedstatement)
  - [DatabaseMetaData](#databasemetadata)

---

# 简介

JDBC（Java DataBase Connectivity, Java 数据库连接）是一种用于执行 SQL 语句的 Java API，可以为多种关系数据库提供统一访问，它由一组用 Java 语言编写的类和接口组成。JDBC 提供了一种基准，据此可以构建更高级的工具和接口，使数据库开发人员能够编写数据库应用程序。

JDBC 提供了一套规范，这样帮助我们不用去关心特定数据库厂商的实现逻辑，只需按照 JDBC 规范编写代码即可

简单的说，JDBC 可以做三件事：
1. 与数据库建立链接
1. 发送、操作数据库的语句
1. 处理结果

# 使用 `JDBC` 完成一个简单的数据库操作
我们先看看我们最熟悉也是最基础的通过JDBC查询数据库数据，一般需要以下七个步骤：
 
1. 加载数据库驱动
2. 创建并获取数据库链接(Connection)
3. 创建statement对象(Statement或PreparedStatement)
4. 设置sql语句
5. 通过statement执行sql并获取结果
6. 对sql执行结果进行解析处理
7. 释放资源(resultSet、statement、connection)

> 数据库驱动由各个数据库厂商实现

参考代码

```java
package com.zyndev.jdbc;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.Statement;
import java.sql.ResultSet;
import java.sql.SQLException;


public class JdbcTest {
    public static void main(String[] args) {
        //数据库连接
        Connection connection = null;
        //预编译的Statement，使用预编译的Statement提高数据库性能
        Statement statement = null;
        //结果集
        ResultSet resultSet = null;

        try {
            //加载数据库驱动
            Class.forName("com.mysql.jdbc.Driver");

            //通过驱动管理类获取数据库链接
            connection =  DriverManager.getConnection("jdbc:mysql://127。0.0.1:3306/test?characterEncoding=utf-8", "root", "root");
            //定义sql语句 ?表示占位符
            String sql = "select id,name from user";
            //获取预处理statement
            statement = connection.createStatement(sql);
            //向数据库发出sql执行查询，查询出结果集
            resultSet =  statement.executeQuery();
            //遍历查询结果集
            while(resultSet.next()){
                System.out.println(resultSet.getString("id")+"  "+resultSet.getString("name"));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            if(resultSet!=null){
                try {
                    resultSet.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if(preparedStatement!=null){
                try {
                    preparedStatement.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if(connection!=null){
                try {
                    connection.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

```

# 关于事务

## 概念
例如：在关系数据库中，一个事务可以是一条SQL语句，一组SQL语句或整个程序。
## 特性
事务是恢复和并发控制的基本单位。

事务应该具有4个属性：原子性、一致性、隔离性、持久性。这四个属性通常称为ACID特性。

- 原子性（atomicity）。一个事务是一个不可分割的工作单位，事务中包括的诸操作要么都做，要么都不做。
- 一致性（consistency）。事务必须是使数据库从一个一致性状态变到另一个一致性状态。一致性与原子性是密切相关的。
- 隔离性（isolation）。一个事务的执行不能被其他事务干扰。即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰。
- 持久性（durability）。持久性也称永久性（permanence），指一个事务一旦提交，它对数据库中数据的改变就应该是永久性的。接下来的其他操作或故障不应该对其有任何影响。
## 提交和回滚

将一条或一组语句构建成一个事务。当所有语句全部成功完成，事务可以被提交。否则，如果其中某个语句遇到错误，那么事务将被回滚，就好像没有执行过任何语句一样。

默认情况下，MySQL数据库链接处于自动提交状态，也就是每一条都会当成一个事务进行操作。当我们需要将一组语句作为一个事务时，这时我们需要关闭它的自动提交功能。
## 保存点 

使用保存点(save point)，可以更好的控制事务的回滚操作。创建一个保存点意味着稍后只需返回到这个点，而非事务的开头。

# API 参考

## Connection
代表一个数据库连接

**常用方法**
- Statement createStatement();
	
	创建一个Statement对象
- PreparedStatement prepareStatement(String sql);
	
	根据SQL出创建一个PreparedStatement对象
- CallableStatement prepareCall(String sql);
	
	创建一个CallableStatement对象来调用数据库存储过程
- setAutoCommit(boolean transaction);
	
	如果该值为 false 则开启手动事务,需要手动执行 commit 才会提交
- commit();
	
	提交事务
- rollback();
	
	回滚事务
## Statement
是一个接口,可以使用它来执行SQL语句
 
**实例方法**

- boolean execute(String sql)
	
	执行SQL
- ResultSet executeQuery(String sql);
	
	执行给定的 SQL 检索 语句，该语句返回单个 ResultSet 对象。
- int executeUpdate(String sql)
	
	执行给定 SQL 语句，该语句可能为 INSERT、UPDATE 或 DELETE 语句，或者不返回任何内容的 SQL 语句（如 SQL DDL 语句）,返回受影响的行数
## ResultSet

SQL执行结果集
**实例方法**

- first()
	
	将光标移动到此 ResultSet 对象的第一行。
- next();
	
	指针移到下一行,如果有下一行返回 true
- previous();
	
	指针移到上一行
- getObject(int columnIndex);
	
	获取指定 index 列的数据,以Object形式返回
- getObject(String columnLabel);
	
	获取指定 列名称 的数据,以Object形式返回
- getString(String columnName);
	
	获取指定字段的String类型值
- getString(int columnIndex);
	
	获取指定索引的String类型值
	
## PreparedStatement

预编译 SQL 的 statement,可以防止SQL注入

**实例方法**

- setString(int index, String param);
	
	给第index个问号赋值，有N多重载(setXxx),可以赋值不同的数据类型，注意'?'号的角标是从1开始,而不是0
- boolean execute()
	
	执行SQL语句该语句可以是任何种类的 SQL 语句。
- ResultSet executeQuery();
	
	执行检索,并返回该查询生成的 ResultSet 对象。
- int executeUpdate()
	
	执行插入或者修改语句


## DatabaseMetaData
