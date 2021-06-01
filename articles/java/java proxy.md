# java 代理

代理(Proxy)是一种设计模式,提供了对目标对象另外的访问方式;即通过代理对象访问目标对象.这样做的好处是:可以在目标对象实现的基础上,增强额外的功能操作,即扩展目标对象的功能.<br>
代理模式的关键点是:代理对象与目标对象.代理对象是对目标对象的扩展,并会调用目标对象

<!-- TOC -->

- [java 代理](#java-%e4%bb%a3%e7%90%86)
  - [静态代理](#%e9%9d%99%e6%80%81%e4%bb%a3%e7%90%86)
  - [动态代理](#%e5%8a%a8%e6%80%81%e4%bb%a3%e7%90%86)
    - [JDK中生成代理对象的API](#jdk%e4%b8%ad%e7%94%9f%e6%88%90%e4%bb%a3%e7%90%86%e5%af%b9%e8%b1%a1%e7%9a%84api)

<!-- /TOC -->

## 静态代理

静态代理在使用时,需要定义接口或者父类,被代理对象与代理对象一起实现相同的接口或者是继承相同父类.

下面举个案例来解释:

在项目中活着网上例子最多的一种例子就是日志操作。日志并不属于业务逻辑中，但日志记录能帮我们确定整个系统的运行状态以及运行过程中的一些细节。

下面以一个用户的登录和登出坐例子（这只是一个例子）

代码示例:

接口:ILogin.java
``` java
public interface ILogin {

    void login(String userName, String password);

    void logout(String userName);
}
```
目标对象:LoginController.java
``` java
public class LoginController implements ILogin {

    public void login(String userName, String password) {
        // 用户验证登录操作
    }

    public void logout(String userName) {
        // 用户退出操作
    }
}
```
代理对象:LoginControllerProxy.java
``` java
/**
 * 代理对象,静态代理
 */
public class LoginControllerProxy implements ILogin{
    // 日志
    private static Logger logger = Logger.getLogger(LoginControllerProxy.Class);

    //接收保存目标对象
    private ILogin target;
    public LoginControllerProxy(ILogin target){
        this.target=target;
    }

    public void login(String userName, String password) {
        // 添加验证
        logger.info("用户登录：用户名：" + userName + "\t密码：" + password + "\tdate:" + new Date());
        target.login(userName, password);
    }

    public void logout(String userName) {
        // 用户退出
        logger.info("用户退出：用户名：" + userName + "\tdate:" + new Date());
        target.logout(userName);
    }
}
```
测试类:App.java

``` java
public class App {
    public static void main(String[] args) {
        //目标对象
        LoginController target = new LoginController();

        //代理对象,把目标对象传给代理对象,建立代理关系
        LoginControllerProxy proxy = new LoginControllerProxy(target);

        proxy.login("张瑀楠", "zyndev@gmail.com");
        proxy.logout("张瑀楠");
    }
}
```
静态代理总结:

1.优点：可以做到在不修改目标对象的功能前提下,对目标功能扩展.

2.缺点:因为代理对象需要与目标对象实现一样的接口,所以会有很多代理类,类太多.同时,一旦接口增加方法,目标对象与代理对象都要维护.

_如何解决静态代理中的缺点呢?答案是可以使用动态代理方式_
## 动态代理

动态代理有以下特点:

1. 代理对象,不需要实现接口

2. 代理对象的生成,是利用JDK的API,动态的在内存中构建代理对象(需要我们指定创建代理对象/目标对象实现的接口的类型)

3. 动态代理也叫做:JDK代理,接口代理

### JDK中生成代理对象的API
代理类所在包:java.lang.reflect.Proxy

JDK实现代理只需要使用newProxyInstance方法,但是该方法需要接收三个参数,完整的写法是:
``` java
static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces,InvocationHandler h )
```
注意该方法是在Proxy类中是静态方法,且接收的三个参数依次为:

1. ClassLoader loader:指定当前目标对象使用类加载器,获取加载器的方法是固定的
2. Class<?>[] interfaces:目标对象实现的接口的类型,使用泛型方式确认类型
3. InvocationHandler h:事件处理,执行目标对象的方法时,会触发事件处理器的方法,会把当前执行目标对象的方法作为参数传入

代码示例:
接口类`ILogin.java`以及接口实现类,目标对象`LoginController.java`是一样的,没有做修改.在这个基础上,增加一个代理工厂类(ProxyFactory.java),将代理类写在这个地方,然后在测试类(需要使用到代理的代码)中先建立目标对象和代理对象的联系,然后代用代理对象的中同名方法

代理工厂类:ProxyFactory.java

``` java
/**
 * 创建动态代理对象
 * 动态代理不需要实现接口,但是需要指定接口类型
 */
public class ProxyFactory{

    private static Logger logger = Logger.getLogger(ProxyFactory.Class);

    //维护一个目标对象
    private Object target;

    public ProxyFactory(Object target){
        this.target=target;
    }

    //给目标对象生成代理对象
    public Object getProxyInstance(){
        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        logger.info("调用方法前：" + method.getName + "\targs:" + args);
                        //执行目标对象方法
                        Object returnValue = method.invoke(target, args);
                        logger.info("调用方法后：" + method.getName + "\treturnValue:" + returnValue);
                        return returnValue;
                    }
                }
        );
    }

}
```
测试类:App.java

``` java
public class App {
    public static void main(String[] args) {
        //目标对象
        LoginController target = new LoginController();
        // 给目标对象，创建代理对象
        ILogin proxy = (IUserDao) new ProxyFactory(target).getProxyInstance();
        proxy.login("张瑀楠", "zyndev@gmail.com");
        proxy.logout("张瑀楠");
    }
}
```
相比静态代理:

代理对象不需要实现接口,但是目标对象一定要实现接口,否则不能用动态代理

[使用cglib实现代理](./java-cglib.md)

---
扩展阅读：

[设计模式-代理模式](https://www.ibm.com/developerworks/cn/java/j-lo-proxy-pattern/index.html)