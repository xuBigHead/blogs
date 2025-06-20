---
layout: post
title: 第006章-Spring Bean
categories: [Spring]
description: 
keywords: Spring Bean.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# Spring_Beans

Spring beans 是那些形成Spring应用的主干的java对象。它们被Spring IOC容器初始化，装配，和管理。这些beans通过容器中配置的元数据创建。比如，以XML文件中 的形式定义。

一个Spring Bean 的定义包含容器必知的所有配置元数据，包括如何创建一个bean，它的生命周期详情及它的依赖。



## 概念

### Bean相关

#### 内部Bean

在Spring框架中，当一个bean仅被用作另一个bean的属性时，它能被声明为一个内部bean。内部bean可以用setter注入“属性”和构造方法注入“构造参数”的方式来实现，内部bean通常是匿名的，它们的Scope一般是prototype。



#### Bean装配

装配，或bean 装配是指在Spring 容器中把bean组装到一起，前提是容器需要知道bean的依赖关系，如何通过依赖注入来把它们装配到一起。



## 对象自动装配

概念

在Spring框架中，在配置文件中设定bean的依赖关系是一个很好的机制，Spring 容器能够自动装配相互合作的bean，这意味着容器不需要和配置，能通过Bean工厂自动处理bean之间的协作。这意味着 Spring可以通过向Bean Factory中注入的方式自动搞定bean之间的依赖关系。自动装配可以设置在每个bean上，也可以设定在特定的bean上。

在spring中，对象无需自己查找或创建与其关联的其他对象，由容器负责把需要相互协作的对象引用赋予各个对象，使用autowire来配置自动装载模式。



自动装配类型

在Spring框架xml配置中共有5种自动装配：

- no：默认的方式是不进行自动装配的，通过手工设置ref属性来进行装配bean。
- byName：通过bean的名称进行自动装配，如果一个bean的 property 与另一bean 的name 相同，就进行自动装配。
- byType：通过参数的数据类型进行自动装配。
- constructor：利用构造函数进行装配，并且构造函数的参数通过byType进行装配。
- autodetect：自动探测，如果有构造方法，通过 construct的方式自动装配，否则使用 byType的方式自动装配。





### @Autowired注解

使用@Autowired注解来自动装配指定的bean。在使用@Autowired注解之前需要在Spring配置文件进行配置，<context:annotation-config />。

在启动spring IoC时，容器自动装载了一个AutowiredAnnotationBeanPostProcessor后置处理器，当容器扫描到@Autowied、@Resource或@Inject时，就会在IoC容器自动查找需要的bean，并装配给该对象的属性。在使用@Autowired时，首先在容器中查询对应类型的bean：

- 如果查询结果刚好为一个，就将该bean装配给@Autowired指定的数据；
- 如果查询的结果不止一个，那么@Autowired会根据名称来查找；
- 如果上述查找的结果为空，那么会抛出异常。解决方法时，使用required=false。



## 对象配置方式

### 注解方式

一般使用 `@Autowired` 注解自动装配 bean，要想把类标识成可用于 `@Autowired` 注解自动装配的 bean 的类，采用以下注解可实现：

- `@Component` ：通用的注解，可标注任意类为 `Spring` 组件。如果一个Bean不知道属于哪个层，可以使用`@Component` 注解标注。
- `@Repository` : 对应持久层即 Dao 层，主要用于数据库相关操作。
- `@Service` : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao层。
- `@Controller` : 对应 Spring MVC 控制层，主要用户接受用户请求并调用 Service 层返回数据给前端页面。



### XML配置文件方式

Spring配置文件是个XML 文件，这个文件包含了类信息，描述了如何配置它们，以及如何相互调用。



### 基于Java代码方式



## 对象作用域

当定义一个 在Spring里，我们还能给这个bean声明一个作用域。它可以通过bean 定义中的scope属性来定义。如，当Spring要在需要的时候每次生产一个新的bean实例，bean的scope属性被指定为prototype。另一方面，一个bean每次使用的时候必须返回同一个实例，这个bean的scope 属性 必须设为 singleton。

| 作用域            | 描述                 | 适用场景 |
| ----------------- | -------------------- | -------- |
| singleton（默认） | 单例                 | 所有场景 |
| prototype         | 多例                 | 所有场景 |
| request           | 在一次http请求内有效 | Web场景  |
| session           | 在一个用户会话内有效 | Web场景  |
| globalSession     | 在全局会话内有效     | Web场景  |



### singleton

bean在每个Spring ioc 容器中只有一个实例。

Springboot的注入默认范围是单例，在容器中通过对象引入配置注入和通过容器的getBean()方法返回的实例都是同一个bean。容器在启动的时候，自动实例化所有的singleton的bean并缓存于容器当中。

1、对bean提前的实例化操作，会及早发现一些潜在的配置的问题。

2、Bean以缓存的方式运行，当运行到需要使用该bean的时候，就不需要再去实例化了。加快了运行效率。



#### 线程安全问题

单例 bean 存在线程问题，主要是因为当多个线程操作同一个对象的时候，对这个对象的非静态成员变量的写操作会存在线程安全问题。

常见解决办法：

1. 在Bean对象中尽量避免定义可变的成员变量。
2. 在类中定义一个ThreadLocal成员变量，将需要的可变成员变量保存在 ThreadLocal 中（推荐的一种方式），注意使用完成后要remove掉。
3. 使用并发安全的类
4. 改变为非单例模式，通过注解 @`Scope("prototype")` 或 `@Scope("request")`



不是，Spring框架中的单例bean不是线程安全的。

spring 中的 bean 默认是单例模式，spring 框架并没有对单例 bean 进行多线程的封装处理。

实际上大部分时候 spring bean 无状态的（比如 dao 类），所有某种程度上来说 bean 也是安全的，但如果 bean 有状态的话（比如 view model 对象），那就要开发者自己去保证线程安全了，最简单的就是改变 bean 的作用域，把“singleton”变更为“prototype”，这样请求 bean 相当于 new Bean()了，所以就可以保证线程安全了。

- 有状态就是有数据存储功能。
- 无状态就是不会保存数据。



在一般情况下，只有无状态的Bean才可以在多线程环境下共享，在Spring中，绝大部分Bean都可以声明为singleton作用域，因为Spring对一些Bean中非线程安全状态采用ThreadLocal进行处理，解决线程安全问题。



#### 实现原理

> org.springframework.beans.factory.support.AbstractBeanFactory

```java
// 一级缓存
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
                          @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
	// ...
    if (mbd.isSingleton()) {
        sharedInstance = getSingleton(beanName, () -> {
            try {
                return createBean(beanName, mbd, args);
            } catch (BeansException ex) {
                destroySingleton(beanName);
                throw ex;
            }
        });
        bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
    }
	// ...
	return (T) bean;
}

public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
    synchronized (this.singletonObjects) {
        Object singletonObject = this.singletonObjects.get(beanName);
        if (singletonObject == null) {
            // ...
            try {
                singletonObject = singletonFactory.getObject();
                newSingleton = true;
            }
            catch (IllegalStateException ex) {
                singletonObject = this.singletonObjects.get(beanName);
                if (singletonObject == null) {
                    throw ex;
                }
            }
            // ...
            finally {
                // ...
                afterSingletonCreation(beanName);
            }
            if (newSingleton) {
                addSingleton(beanName, singletonObject);
            }
        }
        return singletonObject;
    }
}

protected void addSingleton(String beanName, Object singletonObject) {
    synchronized (this.singletonObjects) {
        this.singletonObjects.put(beanName, singletonObject);
        this.singletonFactories.remove(beanName);
        this.earlySingletonObjects.remove(beanName);
        this.registeredSingletons.add(beanName);
    }
}
```



获取bean时判断如果scope是singleton，则调用getSingleton方法获取实例。getSingleton方法的主要逻辑如下：

1. 根据beanName先从singletonObjects集合中获取bean实例。
2. 如果bean实例不为空，则直接返回该实例。
3. 如果bean实例为空，则通过getObject方法创建bean实例，然后通过addSingleton方法，将该bean实例添加到singletonObjects集合中。
4. 下次再通过beanName从singletonObjects集合中，就能获取到bean实例了。



### prototype

一个bean的定义可以有多个实例。

是指每次从容器中调用Bean时，都返回一个新的实例，即每次调用getBean()时，相当于执行new Bean()的操作。在默认情况下，容器在启动时不实例化prototype的Bean，容器将prototype的实例交给调用者之后便不在管理他的生命周期了。

对于有状态的Bean应该使用prototype，对于无状态的Bean则使用singleton

使用 prototype 作用域需要慎重的思考，因为频繁创建和销毁 bean 会带来很大的性能开销。



### request

每次http请求都会创建一个bean，该作用域仅在基于web的Spring ApplicationContext情形下有效。

对应一个http请求和生命周期，当http请求调用作用域为request的bean的时候,Spring便会创建一个新的bean，在请求处理完成之后便及时销毁这个bean。



### session

在一个HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。

Session中所有http请求共享同一个请求的bean实例。Session结束后就销毁bean。



### globalSession

在一个全局的HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。

与session大体相同，但仅在portlet应用中使用。Portlet规范定义了全局session的概念。请求的bean被组成所有portlet的自portlet所共享。如果不是在portlet这种应用下，globalSession则等价于session作用域。





## FactoryBean

一般情况下，Spring通过反射机制利用<bean>的class属性指定实现类实例化Bean，在某些情况下，实例化Bean过程比较复杂，如果按照传统的方式，则需要在<bean>中提供大量的配置信息。配置方式的灵活性是受限的，这时采用编码的方式可能会得到一个简单的方案。Spring为此提供了一个org.springframework.bean.factory.FactoryBean的工厂类接口，用户可以通过实现该接口定制实例化Bean的逻辑。FactoryBean接口对于Spring框架来说占用重要的地位，Spring自身就提供了70多个FactoryBean的实现。

```java
@Service
public class DefaultFactoryBean implements FactoryBean<SchoolBean> {
    private String name = "default factory bean";

    @Override
    public SchoolBean getObject() {
        return new SchoolBean(1L, "第一中学");
    }

    @Override
    public Class<?> getObjectType() {
        return SchoolBean.class;
    }
}
```



### 源码解析

```java
public interface FactoryBean<T> {
    String OBJECT_TYPE_ATTRIBUTE = "factoryBeanObjectType";
    
    /**
     * 返回由FactoryBean创建的Bean实例，
     * 如果isSingleton()返回true，则该实例会放到Spring容器中单实例缓存池中；
     */
    @Nullable
    T getObject() throws Exception;
    
    /**
     * 返回FactoryBean创建的Bean类型。
     */
    @Nullable
    Class<?> getObjectType();
    
    /**
     * 返回由FactoryBean创建的Bean实例的作用域是singleton还是prototype
     */
    default boolean isSingleton() {
        return true;
    }
}
```



FactoryBean 接口提供三个方法，用来创建对象，FactoryBean 具体返回的对象是由getObject 方法决定的。



### 扩展

#### BeanFactory 和 FactoryBean 的区别

BeanFactory 是 Bean 的工厂， ApplicationContext 的父类，IOC 容器的核心，负责生产和管理 Bean 对象。

FactoryBean 是 Bean，可以通过实现 FactoryBean 接口定制实例化 Bean 的逻辑，通过代理一个Bean对象，对方法前后做一些操作。



`BeanFactory` 是接口，提供了IOC容器最基本的形式，给具体的IOC容器的实现提供了规范；

`FactoryBean` 也是接口，为IOC容器中Bean的实现提供了更加灵活的方式，`FactoryBean` 在IOC容器的基础上给Bean的实现加上了一个简单工厂模式和装饰模式，可以在getObject()方法中灵活配置。

```java
@Service
public class DefaultBeanFactory implements BeanFactoryAware {
    private BeanFactory beanFactory;

    @Override
    public void setBeanFactory(@NonNull BeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
    }

    public void execute() {
        String beanName = "defaultFactoryBean";
        Object bean = beanFactory.getBean(beanName);
        if(bean instanceof SchoolBean) {
            System.err.println("bean类型是SchoolBean");
        }
        Object factoryBean = beanFactory.getBean(BeanFactory.FACTORY_BEAN_PREFIX + beanName);
        if(factoryBean instanceof FactoryBean) {
            System.err.println("bean类型是FactoryBean");
        }
    }
}
```





## Aware相关

### BeanNameAware

触发点在bean的初始化之前，也就是`postProcessBeforeInitialization`之前。可以扩展这个点，在初始化bean之前拿到spring容器中注册的的beanName，来自行修改这个beanName的值。



> org.springframework.beans.factory.BeanNameAware

```java
public interface BeanNameAware extends Aware {
	void setBeanName(String name);
}
```



```java
public class NormalBeanA implements BeanNameAware{
    public NormalBeanA() {
        System.out.println("NormalBean constructor");
    }

    @Override
    public void setBeanName(String name) {
        System.out.println("[BeanNameAware] " + name);
    }
}
```