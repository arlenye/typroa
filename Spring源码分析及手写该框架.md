## Spring源码分析及手写该框架



![1532252226394](1532252226394.png)



Spring MVC： 把每一个URL映射到某一个类的某一个方法，传参自动ORM，简化开发。



```java
public interface BeanFactory {

	String FACTORY_BEAN_PREFIX = "&";

	Object getBean(String name) throws BeansException;

	<T> T getBean(String name, Class<T> requiredType) throws BeansException;

	<T> T getBean(Class<T> requiredType) throws BeansException;

	Object getBean(String name, Object... args) throws BeansException;

	<T> T getBean(Class<T> requiredType, Object... args) throws BeansException;

	boolean containsBean(String name);

	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;

	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;

	boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;

	boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;

	Class<?> getType(String name) throws NoSuchBeanDefinitionException;
	
	String[] getAliases(String name);
```



定位、加载、注册。



#### IOC容器：

1. 定位资源（定位查找配置文件）

2. 加载（已经找到配置文件）

3. 注册（已经加载好配置文件解析出来，并封装成BeanDefinition）对bean的说明而已，bean还没有真正产生

   

   #### Bean什么时候初始化呢？

    在获取bean要使用的时候初始化，例如ApplicationContext.getBean();

#### DI 依赖注入：

第一次通过getBean方法想IOC索要bean的时候，IOC容器触发依赖注入。

用户在bean定义资源中为bean元素配置了lazy-init属性，容器在解析注册bean的时候，进行预实例化，触发依赖注入。



#### BeanFactory和factoryBean

BeanFactory是IOC容器的编程抽象比如Alicationcontext,XmlBeanFactory等，是一个factory，生产bean的。FactoryBean是一个可以在IOC容器中被管理的一个bean，是对各种处理过程和资源使用的抽象，Factorybean使用额时候，返回他生产的bean

FACTORY_BEAN_PREFIX=“&”   



#### 实例化bean

AbstractBeanFactory.getBean()

1. 判断是不是单例
2. 判断依注入如中有没有单例，如果有，使用，没有要注入一个

spring默认用cglib代理生成bean，好处，拥有这个代理类的控制权，方便做切面

 AbstractBeanFactory -- AbstractAutowireBeanFactory--SimpleInstantian--



1. 获取BeanDefinition的信息
2. 调用factory的createBean方法
3. createBeanInstance（）可能用jdk的代理，也可能用cglib的代理，依赖关系有List Array Map等。
4. populateBean注入，要做类型转换
5. 放到FactoryBeanObjectCache,真正的IOC容器

