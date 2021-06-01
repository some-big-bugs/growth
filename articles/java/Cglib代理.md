# Cglib代理

Java 提供的静态代理和动态代理模式都是要求目标对象实现一个接口，但是有时候目标对象只是一个单独的对象(例如`hibernate`中的`model`活着`springMVC`中的`controller`)，并没有实现任何的接口，这个时候就可以使用以目标对象子类的方式实现搭理，其中实现的方法就是:Cglib代理

Cglib代理,也叫作子类代理,它是在内存中构建一个子类对象从而实现对目标对象功能的扩展.

JDK的动态代理有一个限制,就是使用动态代理的对象必须实现一个或多个接口,如果想代理没有实现接口的类,就可以使用Cglib实现.

Cglib是一个强大的高性能的代码生成包,它可以在运行期扩展java类与实现java接口.它广泛的被许多AOP的框架使用,例如`Spring AOP`和`synaop`,为他们提供方法的interception(拦截)

Cglib包的底层是通过使用一个小而快的字节码处理框架`ASM`来转换字节码并生成新的类.不鼓励直接使用ASM,因为它要求你必须对JVM内部结构包括`class`文件的格式和指令集都很熟悉.

Cglib子类代理实现方法:
1. 需要引入cglib的jar文件,但是Spring的核心包中已经包括了Cglib功能,所以直接引入`spring-core.jar`即可.
2. 引入功能包后,就可以在内存中动态构建子类
3. 代理的类不能为final,否则报错
4. 目标对象的方法如果为final/static,那么就不会被拦截,即不会执行目标对象额外的业务方法.

代码示例:
目标对象类:UserDao.java
``` java
/**
 * 目标对象,没有实现任何接口
 */
public class LoginController implements ILogin {

    public void login(String userName, String password) {
        // 用户验证登录操作
    }

    public void logout(String userName) {
        // 用户退出操作
    }
}
```
Cglib代理工厂:ProxyFactory.java

``` java
public class ProxyFactory implements MethodInterceptor{
    //维护目标对象
    private Object target;

    public ProxyFactory(Object target) {
        this.target = target;
    }

    //给目标对象创建一个代理对象
    public Object getProxyInstance(){
        //1.工具类
        Enhancer en = new Enhancer();
        //2.设置父类
        en.setSuperclass(target.getClass());
        //3.设置回调函数
        en.setCallback(this);
        //4.创建子类(代理对象)
        return en.create();

    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("开始事务...");

        //执行目标对象的方法
        Object returnValue = method.invoke(target, args);

        System.out.println("提交事务...");

        return returnValue;
    }
}
```
测试类:
``` java
public class App {

    @Test
    public void test(){
        //目标对象
        UserDao target = new UserDao();

        //代理对象
        UserDao proxy = (UserDao)new ProxyFactory(target).getProxyInstance();

        //执行代理对象的方法
        proxy.save();
    }
}
```
在Spring的AOP编程中:<br>
如果加入容器的目标对象有实现接口,用JDK代理<br>
如果目标对象没有实现接口,用Cglib代理

