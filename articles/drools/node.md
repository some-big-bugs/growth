![image-20210425113730735](/Users/zhangyunan/project/growth/img/image-20210425113730735.png)



# drools 支持的运算条件



| 类型           | 关键字                                 | 语法规则                                       | 说明 |
| -------------- | -------------------------------------- | ---------------------------------------------- | ---- |
| 基本运算符     | `+`、 `-` 、 `*`、 `/`、 `%`           |                                                |      |
| 约束符的连接符 | “&&”（and）、“\|\|”（or）和“，”（and） |                                                |      |
| 比较操作符     | >或<，>=或<=，==或 !=                  |                                                |      |
| 包含比较运算符 | `contains` 、`not contains`            |                                                |      |
| 成员比较运算符 | `memberOf` 、`not memberOf`            |                                                |      |
| 正则匹配运算符 | `matches`、`not matches`               | `fieldname matches | not matches "正则表达式"` |      |
|                | soundslike                             |                                                |      |
|                |                                        |                                                |      |



硬关键字主要包括true、false、null。编写规则时，一定要注意软关键字不像硬关键字那么强制，软关键字相比硬关键字要多，如果非要使用软关键字作为命名是没有问题的。软关键字包含lock-on-active、date-effective、date-expires、no-loop、auto-focus、activation-group、agenda-group、ruleflow-group、entry-point、duration、package、import、dialec、salience、enabled、attributes、rule、extend、template、query、declare、function、global、eval、not、in、or、and、exists、forall、action、reverse、result、end、init等



# 执行指定的规则



可以在 `KieSession#fireAllRules`指定规则过滤器来过滤规则

原执行代码为：

```java
KieServices kss = KieServices.Factory.get();
KieContainer kc = kss.getKieClasspathContainer();
KieSession ks = kc.newKieSession("person");
ks.fireAllRules();
ks.dispose();
```



改之后为：

```java
KieServices kss = KieServices.Factory.get();
KieContainer kc = kss.getKieClasspathContainer();
KieSession ks = kc.newKieSession("person");
ks.fireAllRules(new RuleNameEqualsAgendaFilter("规则名称"));
ks.dispose();
```



其中 `AgendaFilter` drools 默认提供了 4 种，分别是

- RuleNameEndsWithAgendaFilter   以指定后缀结尾的
- RuleNameEqualsAgendaFilter        以指定名称的
- RuleNameMatchesAgendaFilter     符合正则的
- RuleNameStartsWithAgendaFilter  以指定前缀开始的

如果想自定义其他的过滤规则，可以实现 `org.kie.api.runtime.rule.AgendaFilter` 接口即可

```java
package org.kie.api.runtime.rule;

public interface AgendaFilter {

    /**
     * Determine if a given match should be fired.
     *
     * @param match
     *     The match that is requested to be fired
     * @return
     *     boolean value of "true" accepts the match for firing.
     */
    boolean accept(Match match);
}
```



`KieSession#fireAllRules` 调用方法做以下总结: 

- int fireAllRules()  执行所有满足条件的规则。
- int fireAllRules(int max)  执行规则的最大数量，简单地说，即执行多少条规则。
- int fireAllRules(AgendaFilter agendaFilter)  指定规则名的方式。
- int fireAllRules(AgendaFilter agendaFilter，int max)  指定规则名，并设置执行规则的最大数量，在精确匹配中是不适用的。



# 动态规则

所谓动态规则，是指在不重启服务器的前提下使业务规则发生变化，且不影响服务器的正常使用，从而实现动态业务变化的规则。这也是规则引擎的魅力所在。

规则引擎要实现动态规则并不容易，动态规则可分为以下7种方式。

- （1）字符串方式，拼接规则语法，形成完成的规则内容，通过string方式调用规则。
- （2）通过规则模板方式对规则进行修改。
- （3）Workbench 自动扫描。
- （4）Workbench 整合Kie-Server。
- （5）指定文件的方式，通过规则文件执行规则。
- （6）通过代码生成kjar。



**动态规则的两种主流**

- 第一种：Workbench+Kie-Server集群+负载均衡。

Workbench+Kie-Server方式，Workbench是定位，WEB是IDE，通过这类IDE都可以完成动态规则的实现，这种方式是程序员相对喜欢的方式。

- 第二种：String+DB的应用。

String+DB规则内容入库方式，操作数据库，实现定制化，作为Web项目，操作数据库是正常不过的，这里通过页面进行配置，生成规则，业务人员这样的技术小白自定制规则，更适合互联网这样的项目应用。例如，金融系统，它的业务变化快，效果要求高。但业务开发一套“类似“Workbench的页面操作规则系统，注意点比较多，页面的设置、数据表的设计等。重要的是，语法的测试及业务场景的测试都是要有的，因此真正实现起来也是非常困难的。



# 规则语法的整合优化

规则在执行时，会有一个类似预处理的过程，规则只执行满足条件的部分（如果使用accumulate可以测试），而并不是因为规则多，其实规则多少对整合性能影响并不是很大，只是在生成规则库时需要大量的时间。所以优化规则语法的原则如下。

- （1）规则拆分原则：将规则进行拆分，避免出现OR的情况。
- （2）规则比较原则：将区间或模糊查询的方式排在比较值的后面。
- （3）规则简单原则：尽量避免出现过于复杂的比较值。
- （4）规则结果原则：then中避免出来if eles。



![image-20210426155721347](/Users/zhangyunan/project/growth/img/image-20210426155721347.png)





规则引擎在项目中的位置是设计人员必须清楚的，规则引擎并不是万能的，被使用的原因是它可以让业务更透明、简单，解放生产力，而一个完整的项目体系可远远不止这些。



决策层中包含了模型管理、规则配置、知识库、规则库、规则调用、业务管理。通过此图要说明的问题是，规则引擎是整合项目中的一个部分，它是需要结合其他服务来协作的，如权限管理，业务操作员受权限的影响，其操作的范围也就有所不同。所以定位决定了设计的范围。



![image-20210426160821776](/Users/zhangyunan/project/growth/img/image-20210426160821776.png)





