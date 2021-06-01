# Apollo 是如何实现配置更新的

> 这篇文档主要关注下配置修改后对应的 Java 对象是如何更新，并不关注整体的配置改动流程
>
> 所有代码都来自 apollo-client 项目

# 更新流程

在 Apollo 控制台进行配置修改并发布后，对应的 client 端拉取到更新后，会调用到 `com.ctrip.framework.apollo.spring.property.AutoUpdateConfigChangeListener#onChange` 方法

在调用 onChange 会收到对应的修改的配置信息 ConfigChangeEvent， 其中包含改动的 key 和 value, 则改动流程如下：

1. 根据改动的配置的 key 从 springValueRegistry 找到对应的关联到这个 key 的 Spring Bean 信息，如果找不到则不处理
2. 根据找到的 Spring Bean 信息，进行对应关联配置的更新


> 在第二步中会判断关联配置是用过属性关联还是方法进行关联的，代码如下
>
> ```java
> public void update(Object newVal) throws IllegalAccessException, InvocationTargetException {
>     if (isField()) {
>        injectField(newVal);
>     } else {
>        injectMethod(newVal);
>     }
> }
> ```

在上面的问题中，还有两个问题存疑

1. 如何通过 key 找到对应的 Spring Bean 信息
2. 如何将 Apollo 的配置值转换为 Spring 的识别的值 

```java
public class AutoUpdateConfigChangeListener implements ConfigChangeListener{
  private static final Logger logger = LoggerFactory.getLogger(AutoUpdateConfigChangeListener.class);

  private final boolean typeConverterHasConvertIfNecessaryWithFieldParameter;
  private final Environment environment;
  private final ConfigurableBeanFactory beanFactory;
  private final TypeConverter typeConverter;
  private final PlaceholderHelper placeholderHelper;
  private final SpringValueRegistry springValueRegistry;
  private final Gson gson;

  public AutoUpdateConfigChangeListener(Environment environment, ConfigurableListableBeanFactory beanFactory){
    this.typeConverterHasConvertIfNecessaryWithFieldParameter = testTypeConverterHasConvertIfNecessaryWithFieldParameter();
    this.beanFactory = beanFactory;
    this.typeConverter = this.beanFactory.getTypeConverter();
    this.environment = environment;
    this.placeholderHelper = SpringInjector.getInstance(PlaceholderHelper.class);
    this.springValueRegistry = SpringInjector.getInstance(SpringValueRegistry.class);
    this.gson = new Gson();
  }

  @Override
  public void onChange(ConfigChangeEvent changeEvent) {
    Set<String> keys = changeEvent.changedKeys();
    if (CollectionUtils.isEmpty(keys)) {
      return;
    }
    for (String key : keys) {
      // 1. check whether the changed key is relevant
      Collection<SpringValue> targetValues = springValueRegistry.get(beanFactory, key);
      if (targetValues == null || targetValues.isEmpty()) {
        continue;
      }

      // 2. update the value
      for (SpringValue val : targetValues) {
        updateSpringValue(val);
      }
    }
  }

  private void updateSpringValue(SpringValue springValue) {
    try {
      Object value = resolvePropertyValue(springValue);
      springValue.update(value);

      logger.info("Auto update apollo changed value successfully, new value: {}, {}", value,
          springValue);
    } catch (Throwable ex) {
      logger.error("Auto update apollo changed value failed, {}", springValue.toString(), ex);
    }
  }

  /**
   * Logic transplanted from DefaultListableBeanFactory
   * @see org.springframework.beans.factory.support.DefaultListableBeanFactory#doResolveDependency(org.springframework.beans.factory.config.DependencyDescriptor, java.lang.String, java.util.Set, org.springframework.beans.TypeConverter)
   */
  private Object resolvePropertyValue(SpringValue springValue) {
    // value will never be null, as @Value and @ApolloJsonValue will not allow that
    Object value = placeholderHelper
        .resolvePropertyValue(beanFactory, springValue.getBeanName(), springValue.getPlaceholder());

    if (springValue.isJson()) {
      value = parseJsonValue((String)value, springValue.getGenericType());
    } else {
      if (springValue.isField()) {
        // org.springframework.beans.TypeConverter#convertIfNecessary(java.lang.Object, java.lang.Class, java.lang.reflect.Field) is available from Spring 3.2.0+
        if (typeConverterHasConvertIfNecessaryWithFieldParameter) {
          value = this.typeConverter
              .convertIfNecessary(value, springValue.getTargetType(), springValue.getField());
        } else {
          value = this.typeConverter.convertIfNecessary(value, springValue.getTargetType());
        }
      } else {
        value = this.typeConverter.convertIfNecessary(value, springValue.getTargetType(),
            springValue.getMethodParameter());
      }
    }

    return value;
  }

  private Object parseJsonValue(String json, Type targetType) {
    try {
      return gson.fromJson(json, targetType);
    } catch (Throwable ex) {
      logger.error("Parsing json '{}' to type {} failed!", json, targetType, ex);
      throw ex;
    }
  }

  private boolean testTypeConverterHasConvertIfNecessaryWithFieldParameter() {
    try {
      TypeConverter.class.getMethod("convertIfNecessary", Object.class, Class.class, Field.class);
    } catch (Throwable ex) {
      return false;
    }
    return true;
  }
}
```



# 如何将配置 key 和 Spring Bean 关联起来

在 Spring 常见配置包括 2 种

```java
public class ApiConfig {
  
  	// 1. 直接在 Field 是进行注入
    @Value("${feifei.appId}")
    protected String appId;

    protected String predUrl;

  	// 2. 在方法上进行注入
    @Value("${predUrl}")
    public void setPredUrl(String predUrl) {
        this.predUrl = predUrl;
    }
}
```



在 Apollo 代码中，通过实现 `BeanPostProcessor` 接口来检测所有的Spring Bean 的创建过程，在 Spring Bean 创建的过程中会调用对应的 `org.springframework.beans.factory.config.BeanPostProcessor#postProcessBeforeInitialization` 和 `org.springframework.beans.factory.config.BeanPostProcessor#postProcessAfterInitialization` 方法。



Apollo 通过在 Bean 生成过程中，检测 Bean 类中属性和方法是否存在 `@Value` 注解，如果存在，提出其中的 key, 其处理方法在 `processField` 和 `processMethod` 分别处理 Field 和 Method 中可能出现的 `@Value` 注解。如果存在注解则将对应的信息存到 `SpringValue` 对应 `springValueRegistry` 全局对象中，方便在其它地方可以直接获取。



在属性除了通过 `@Value` 注入，也可以用过 xml 进行配置，在这种情况通过 `processBeanPropertyValues` 方法来处理



通过两种处理方式就可以将 key 和对应的 Spring Bean 信息关联起来



```java
public class SpringValueProcessor extends ApolloProcessor implements BeanFactoryPostProcessor, BeanFactoryAware {

  private static final Logger logger = LoggerFactory.getLogger(SpringValueProcessor.class);

  private final ConfigUtil configUtil;
  private final PlaceholderHelper placeholderHelper;
  private final SpringValueRegistry springValueRegistry;

  private BeanFactory beanFactory;
  private Multimap<String, SpringValueDefinition> beanName2SpringValueDefinitions;

  public SpringValueProcessor() {
    configUtil = ApolloInjector.getInstance(ConfigUtil.class);
    placeholderHelper = SpringInjector.getInstance(PlaceholderHelper.class);
    springValueRegistry = SpringInjector.getInstance(SpringValueRegistry.class);
    beanName2SpringValueDefinitions = LinkedListMultimap.create();
  }

  @Override
  public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory)
      throws BeansException {
    if (configUtil.isAutoUpdateInjectedSpringPropertiesEnabled() && beanFactory instanceof BeanDefinitionRegistry) {
      beanName2SpringValueDefinitions = SpringValueDefinitionProcessor
          .getBeanName2SpringValueDefinitions((BeanDefinitionRegistry) beanFactory);
    }
  }

  @Override
  public Object postProcessBeforeInitialization(Object bean, String beanName)
      throws BeansException {
    if (configUtil.isAutoUpdateInjectedSpringPropertiesEnabled()) {
      super.postProcessBeforeInitialization(bean, beanName);
      processBeanPropertyValues(bean, beanName);
    }
    return bean;
  }


  @Override
  protected void processField(Object bean, String beanName, Field field) {
    // register @Value on field
    Value value = field.getAnnotation(Value.class);
    if (value == null) {
      return;
    }
    Set<String> keys = placeholderHelper.extractPlaceholderKeys(value.value());

    if (keys.isEmpty()) {
      return;
    }

    for (String key : keys) {
      SpringValue springValue = new SpringValue(key, value.value(), bean, beanName, field, false);
      springValueRegistry.register(beanFactory, key, springValue);
      logger.debug("Monitoring {}", springValue);
    }
  }

  @Override
  protected void processMethod(Object bean, String beanName, Method method) {
    //register @Value on method
    Value value = method.getAnnotation(Value.class);
    if (value == null) {
      return;
    }
    //skip Configuration bean methods
    if (method.getAnnotation(Bean.class) != null) {
      return;
    }
    if (method.getParameterTypes().length != 1) {
      logger.error("Ignore @Value setter {}.{}, expecting 1 parameter, actual {} parameters",
          bean.getClass().getName(), method.getName(), method.getParameterTypes().length);
      return;
    }

    Set<String> keys = placeholderHelper.extractPlaceholderKeys(value.value());

    if (keys.isEmpty()) {
      return;
    }

    for (String key : keys) {
      SpringValue springValue = new SpringValue(key, value.value(), bean, beanName, method, false);
      springValueRegistry.register(beanFactory, key, springValue);
      logger.info("Monitoring {}", springValue);
    }
  }


  private void processBeanPropertyValues(Object bean, String beanName) {
    Collection<SpringValueDefinition> propertySpringValues = beanName2SpringValueDefinitions
        .get(beanName);
    if (propertySpringValues == null || propertySpringValues.isEmpty()) {
      return;
    }

    for (SpringValueDefinition definition : propertySpringValues) {
      try {
        PropertyDescriptor pd = BeanUtils
            .getPropertyDescriptor(bean.getClass(), definition.getPropertyName());
        Method method = pd.getWriteMethod();
        if (method == null) {
          continue;
        }
        SpringValue springValue = new SpringValue(definition.getKey(), definition.getPlaceholder(),
            bean, beanName, method, false);
        springValueRegistry.register(beanFactory, definition.getKey(), springValue);
        logger.debug("Monitoring {}", springValue);
      } catch (Throwable ex) {
        logger.error("Failed to enable auto update feature for {}.{}", bean.getClass(),
            definition.getPropertyName());
      }
    }

    // clear
    beanName2SpringValueDefinitions.removeAll(beanName);
  }

  @Override
  public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
    this.beanFactory = beanFactory;
  }
}
```



