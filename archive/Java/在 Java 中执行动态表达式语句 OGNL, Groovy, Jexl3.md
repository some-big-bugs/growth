<h1> 在 Java 中执行动态表达式语句 </h1>

在一些规则集或者工作流项目中，经常会遇到动态解析表达式并执行得出结果的功能。

规则引擎是一种嵌入在应用程序中的组件，它可以将业务规则从业务代码中剥离出来，使用预先定义好的语义规范来实现这些剥离出来的业务规则；规则引擎通过接受输入的数据，进行业务规则的评估，并做出业务决策。

工作流（Workflow），是对工作流程及其各操作步骤之间业务规则的抽象、概括描述。 工作流建模，即将工作流程中的工作如何前后组织在一起的逻辑和规则，在计算机中以恰当的模型表達并对其实施计算。 工作流要解决的主要问题是：为实现某个业务目标，利用计算机在多个参与者之间按某种预定规则自动传递文档、信息或者任务。

---

**Table of Contents**

- [前缀、中缀、后缀表达式(逆波兰表达式)](#前缀中缀后缀表达式逆波兰表达式)
  - [中缀表达式](#中缀表达式)
  - [后缀表达式](#后缀表达式)
  - [前缀表达式](#前缀表达式)
- [OGNL](#ognl)
- [SpEL](#spel)
- [Jexl/Jexl3](#jexljexl3)
- [执行简单的表达式](#执行简单的表达式)
- [Groovy](#groovy)
  - [执行表达式](#执行表达式)
- [扩展](#扩展)
- [参考](#参考)

---

# 前缀、中缀、后缀表达式(逆波兰表达式)

最早接触的表达式解析是在上数据结构的时候，当时课设作业是 “ 做一个简单的四则混合运算语句解析并计算结果 ”，简单说就是计算器。

## 中缀表达式

**将运算符写在两个操作数中间的表达式，称作中缀表达式。**

中缀表达式是我们最熟悉和阅读最容易的表达式

比如：`12 + 34 + 5 * 6 - 30 / 5`

也就是我们常用的数学算式就是用中缀表达式表示的

## 后缀表达式

**将运算符写在两个操作数之后的表达式称作后缀表达式。**

`12 34 + 5 6 * + 30 5 / -`

前缀表达式需要从左往右读，遇到一个运算法，则从左边取 2 个操作数进行运算

从左到右读则可分为

（（12 34 + ）（5 6 * ）+ ）（30 / 5） -

括号只是辅助，实际上没有

## 前缀表达式

**前缀表达式是将运算符写在两个操作数之前的表达式。**

前缀表达式需要从右往左读，遇到一个运算法，则从右边取 2 个操作数进行运算

12 + 34 + 5 * 6 - 30 / 5

`- + + 12 34 * 5 6 / 30 5`



- 中缀：`12 + 34 + 5 * 6 - 30 / 5`
- 后缀：`12 34 + 5 6 * + 30 5 / -`
- 前缀：`- + + 12 34 * 5 6 / 30 5`

# OGNL

OGNL（Object-Graph Navigation Language的简称），对象图导航语言，它是一门表达式语言，除了用来设置和获取Java对象的属性之外，另外提供诸如集合的投影和过滤以及lambda表达式等。

引入依赖

```xml
<!-- https://mvnrepository.com/artifact/ognl/ognl -->
<dependency>
    <groupId>ognl</groupId>
    <artifactId>ognl</artifactId>
    <version>3.2.18</version>
</dependency>
```

```java
MemberAccess memberAccess = new AbstractMemberAccess() {
    @Override
    public boolean isAccessible(Map context, Object target, Member member, String propertyName) {
        int modifiers = member.getModifiers();
        return Modifier.isPublic(modifiers);
    }
};

OgnlContext context = (OgnlContext) Ognl.createDefaultContext(this,
    memberAccess,
    new DefaultClassResolver(),
    new DefaultTypeConverter());

context.put("verifyStatus", 1);
Object expression = Ognl.parseExpression("#verifyStatus == 1");
boolean result =(boolean) Ognl.getValue(expression, context, context.getRoot());
Assert.assertTrue(result);
```

# SpEL

SpEL(Spring Expression Language)，即Spring表达式语言。它是一种类似JSP的EL表达式、但又比后者更为强大有用的表达式语言。

```java
ExpressionParser parser = new SpelExpressionParser();
Expression expression = parser.parseExpression("#verifyStatus == 1");

EvaluationContext context = new StandardEvaluationContext();
context.setVariable("verifyStatus", 1);
boolean result = (boolean) expression.getValue(context);
Assert.assertTrue(result);
```

# Jexl/Jexl3

引入依赖

```xml
<!-- https://mvnrepository.com/artifact/org.apache.commons/commons-jexl3 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-jexl3</artifactId>
    <version>3.1</version>
</dependency>
```

# 执行简单的表达式

```java
JexlEngine jexl = new JexlBuilder().create();
JexlContext jc = new MapContext();
jc.set("verifyStatus", 1);
JexlExpression expression = jexl.createExpression("verifyStatus == 1");
boolean result = (boolean) expression.evaluate(jc);
Assert.assertTrue(result);
```

# Groovy

Groovy 是一个很好的选择，其具备完备的 Groovy 和 Java 语法的解析执行功能。

引入依赖, 这个可以根据需要引入最新版本

```xml
<!-- https://mvnrepository.com/artifact/org.codehaus.groovy/groovy -->
<dependency>
    <groupId>org.codehaus.groovy</groupId>
    <artifactId>groovy</artifactId>
    <version>2.5.6</version>
</dependency>
```

## 执行表达式

```java
Binding binding = new Binding();
binding.setVariable("verifyStatus", 1);
GroovyShell shell = new GroovyShell(binding);
boolean result = (boolean) shell.evaluate("verifyStatus == 1");
Assert.assertTrue(result);
```

# 扩展

经常用 MyBatis 的一定用过动态语句

```xml
<select id="getList" 
    resultMap="UserBaseMap"
    parameterType="com.xx.Param">
    select
        id, invite_code, phone, name
    from user
    where status = 1
    <if test="_parameter != null">
        <if test="inviteCode !=null and inviteCode !='' ">
            and invite_code = #{inviteCode}
        </if>
    </if>
</select>
```

这里我们简化一下

该示例主要为了讲解，不一定好用, 其中 `@if` 与上面的 `<if>` 等效 

```sql
select id, invite_code, phone, name 
from user 
where status = 1 
@if(:inviteCode != null) { and invite_code = :inviteCode }
```

在处理这种 SQL 中，我们可以先用正则，将 `@if` 与 正常语句分割开

```java
List<String> results = StringUtil.matches(sql, "@if([\\s\\S]*?)}");
```

通过这种方式匹配到 `@if(:inviteCode != null) { and invite_code = :inviteCode }`

然后将需要执行计算的表达式与要拼接的 SQL 分离出 

```java

String text = "@if(:inviteCode != null) { and invite_code = :inviteCode }";

List<String> sqlFragment = StringUtil.matches(text, "\\(([\\s\\S]*?)\\)|\\{([\\s\\S]*?)\\}");
```

分离出

- :inviteCode != null
- and invite_code = :inviteCode

其中 `:inviteCode != null` 是需要动态处理的语句，对于  `:inviteCode != null` 我们需要识别出，那些是需要进行复制的变量名称

```java
List<String> sqlFragmentParam = StringUtil.matches(":inviteCode != null", "\\?\\d+(\\.[A-Za-z]+)?|:[A-Za-z0-9]+(\\.[A-Za-z]+)?");
```

得到 inviteCode，并通过某种方式找到对应的值，


```java
JexlEngine jexl = new JexlBuilder().create();
JexlContext jc = new MapContext();
jc.set(":inviteCode", "ddddsdfa");
JexlExpression expression = jexl.createExpression(sqlExp);
boolean needAppendSQL = (boolean) expression.evaluate(jc);
```

通过 needAppendSQL 来决定是否拼接 SQL, 这样一个简单的动态 SQL 就实现了，上面用的 Jexl 写的，你可以改成上面任意一种方案，这里只做演示

**具体代码，仅供参考**

```java
@Test
public void testSQL() {
  String sql = "select id, invite_code, phone, name \n"
  + "from user \n"
  + "where status = 1 \n"
  + "@if(:inviteCode != null) { and invite_code = :inviteCode }";

  Map<String, Object> params = new HashMap<String, Object>();
params.put("inviteCode", "dd");

  System.out.println(parseJexl(sql, params));
}

public String parseJexl(String jexlSql, Map<String, Object> params) {

  // 判断是否包含 @if
  List<String> results = StringUtil.matches(jexlSql, "@if([\\s\\S]*?)}");
  if (results.isEmpty()) {
      return jexlSql;
  }

  JexlEngine jexl = new JexlBuilder().create();
  JexlContext jc = new MapContext();

  for (String e : results) {
    List<String> sqlFragment = StringUtil.matches(e, "\\(([\\s\\S]*?)\\)|\\{([\\s\\S]*?)\\}");
    String sqlExp = sqlFragment.get(0).trim().substring(1, sqlFragment.get(0).length() - 1);
    List<String> sqlFragmentParam = StringUtil.matches(sqlExp, "\\?\\d+(\\.[A-Za-z]+)?|:[A-Za-z0-9]+(\\.[A-Za-z]+)?");
    for (String param : sqlFragmentParam) {
      String newSQLExp = "_" + param.substring(1);
      sqlExp = sqlExp.replace(param, newSQLExp);
      jc.set(newSQLExp, params.get(param.substring(1)));
    }
    JexlExpression expression = jexl.createExpression(sqlExp);
    Boolean needAppendSQL = (Boolean) expression.evaluate(jc);
    if (needAppendSQL) {
      jexlSql = jexlSql.replace(e, sqlFragment.get(1).trim().substring(1, sqlFragment.get(1).length() - 1));
    } else {
      jexlSql = jexlSql.replace(e, "");
    }
  }
  return jexlSql;
}
```






# 参考

关于 OGNL、SpEL、Jexl3 和 Groovy 的具体用法可以参考文档

- http://commons.apache.org/proper/commons-jexl/reference/syntax.html
- https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#expressions
- http://groovy-lang.org/syntax.html
- http://commons.apache.org/proper/commons-ognl/language-guide.html

