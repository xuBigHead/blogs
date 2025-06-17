---
layout: post
title: 第008章-Spring Bean 循环引用
categories: [Spring]
description: 
keywords: Spring Bean 循环引用.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
## Circular References

循环依赖：说白是一个或多个对象实例之间存在直接或间接的依赖关系，这种依赖关系构成了构成一个环形调用。



### 三级缓存

Spring 解决循环依赖的核心就是提前暴露对象，而提前暴露的对象就是放置于第二级缓存中。下表是三级缓存的说明：

| 名称                  | 描述                                                         |
| --------------------- | ------------------------------------------------------------ |
| singletonObjects      | 一级缓存，存放完整（实例化、初始化都完成）的 Bean。          |
| earlySingletonObjects | 二级缓存，存放提前暴露的不完整（实例化完成，未初始化完成）Bean，未完成属性注入和执行 init 方法。 |
| singletonFactories    | 三级缓存，存放的是 Bean 工厂，主要是生产 Bean，存放到二级缓存中。 |



因为三级缓存中放的是生成具体对象的匿名内部类，他可以生成代理对象，也可以是普通的实例对象。使用三级缓存主要是为了保证不管什么时候使用的都是一个对象。

假设只有二级缓存的情况，往二级缓存中放的显示一个普通的Bean对象，`BeanPostProcessor`去生成代理对象之后，覆盖掉二级缓存中的普通Bean对象，那么多线程环境下可能取到的对象就不一致了。



> org.springframework.beans.factory.support.DefaultSingletonBeanRegistry

```java
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {
	private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
	private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
	private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);
}
```



### 循环依赖场景

循环依赖主要场景如下：

- 单例的setter注入（能解决）；
- 多例的setter注入（不能解决）；
- 构造器注入（不能解决）；
- 单例的代理对象setter注入（有可能解决）；
- DependsOn循环依赖（不能解决）；



#### 单例setter注入

##### 两个Bean互相依赖

```java
@Service
publicclass TestService1 {
    @Autowired
    private TestService2 testService2;
}

@Service
publicclass TestService2 {
    @Autowired
    private TestService1 testService1;
}
```

spring内部有三级缓存：

- singletonObjects 一级缓存，用于保存实例化、注入、初始化完成的bean实例
- earlySingletonObjects 二级缓存，用于保存实例化完成的bean实例
- singletonFactories 三级缓存，用于保存bean创建工厂，以便于后面扩展有机会创建代理对象。

![img](https://oss.xubighead.top/oss/image/202506/1930432950131855362.jpg)

A对象的创建过程：

1. 创建对象A，实例化的时候把A对象工厂放入三级缓存
2. A注入属性时，发现依赖B，转而去实例化B
3. 同样创建对象B，注入属性时发现依赖A，一次从一级到三级缓存查询A，从三级缓存通过对象工厂拿到A，把A放入二级缓存，同时删除三级缓存中的A，此时，B已经实例化并且初始化完成，把B放入一级缓存。
4. 接着继续创建A，顺利从一级缓存拿到实例化且初始化完成的B对象，A对象创建也完成，删除二级缓存中的A，同时把A放入一级缓存
5. 最后，一级缓存中保存着实例化、初始化都完成的A、B对象



##### 多个Bean互相依赖

上述场景中**第二级缓存**作用不大，二级缓存主要用于多个Bean互相依赖的情况。如下所示：



```java
@Service
publicclass TestService1 {
    @Autowired
    private TestService2 testService2;
    @Autowired
    private TestService3 testService3;
}

@Service
publicclass TestService2 {
    @Autowired
    private TestService1 testService1;
}

@Service
publicclass TestService3 {
    @Autowired
    private TestService1 testService1;
}
```



TestService1依赖于TestService2和TestService3，而TestService2依赖于TestService1，同时TestService3也依赖于TestService1。按照上图的流程可以把TestService1注入到TestService2，并且TestService1的实例是从第三级缓存中获取的。

假设不用第二级缓存，TestService1注入到TestService3的流程如图：

![img](https://oss.xubighead.top/oss/image/202506/1930432973989056513.png)



TestService1注入到TestService3又需要从第三级缓存中获取实例，而第三级缓存里保存的并非真正的实例对象，而是`ObjectFactory`对象。**两次从三级缓存中获取都是`ObjectFactory`对象，而通过它创建的实例对象每次可能都不一样的。**



> 第三级缓存中添加`ObjectFactory`对象，而不是直接保存实例对象，主要是为了对添加到三级缓存中的实例对象进行增强。



为了解决这个问题，spring引入的第二级缓存。前一个图其实TestService1对象的实例已经被添加到第二级缓存中了，而在TestService1注入到TestService3时，只用从第二级缓存中获取该对象即可。

![img](https://oss.xubighead.top/oss/image/202506/1930433117463613441.png)



#### 多例的setter注入

```java
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
@Service
publicclass TestService1 {
    @Autowired
    private TestService2 testService2;
}

@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
@Service
publicclass TestService2 {
    @Autowired
    private TestService1 testService1;
}
```



这种情况下虽然是原型场景下的循环依赖，但是程序能够正常启动。因为Spring中非抽象、单例 并且非懒加载的类才能被提前初始bean。而多例即`SCOPE_PROTOTYPE`类型的类，非单例，不会被提前初始化bean，所以程序能够正常启动。



```java
@Service
publicclass TestService3 {
    @Autowired
    private TestService1 testService1;
}
```



如果再定义一个单例的类，在它里面注入TestService1，此时就会要初始化TestService1导致抛出异常。执行结果：

```
Requested bean is currently in creation: Is there an unresolvable circular reference?
```



这种循环依赖问题是无法解决的，因为它没有用缓存，每次都会生成一个新对象。



##### 原型场景

原型(Prototype)的场景是不支持循环依赖的， 单例的场景才能存在循环依赖。原型(Prototype)的场景通常会走到AbstractBeanFactory类中下面的判断，抛出异常。



> org.springframework.beans.factory.support.AbstractBeanFactory

```java
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
			@Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
		// ...
		if (sharedInstance != null && args == null) {
			// ...
		}

		else {
			// 如果已经在创建这个 bean 实例，则失败：可能在循环引用中。
			if (isPrototypeCurrentlyInCreation(beanName)) {
				throw new BeanCurrentlyInCreationException(beanName);
			}
			// ...
			}
		}
```



因为创建新的A时，发现要注入原型字段B，又创建新的B发现要注入原型字段A。这就套娃了, Spring就先抛出了BeanCurrentlyInCreationException。



##### 源码解析

在`AbstractApplicationContext`类的`refresh`方法中会调用`finishBeanFactoryInitialization`方法，该方法的作用是为了spring容器启动的时候提前初始化一些bean。该方法的内部又调用了`preInstantiateSingletons`方法。



> org.springframework.context.support.AbstractApplicationContext

```java
	protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
		// ...
		// 实例化所有剩余的（非惰性初始化）单例
		beanFactory.preInstantiateSingletons();
	}
```



> org.springframework.beans.factory.support.DefaultListableBeanFactory

```java
	public void preInstantiateSingletons() throws BeansException {
		// ...
		for (String beanName : beanNames) {
			RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
			if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
                // 非抽象、单例 并且非懒加载的类才能被提前初始bean。
				// ... 
			}
		}
		// ...
	}
```



#### 构造器注入

```java
@Service
publicclass TestService1 {
    public TestService1(TestService2 testService2) {
    }
}
@Service
publicclass TestService2 {
    public TestService2(TestService1 testService1) {
    }
}
```



运行结果：

```
Requested bean is currently in creation: Is there an unresolvable circular reference?
```



出现了循环依赖，构造器注入没能添加到三级缓存，也没有使用缓存，所以也无法解决循环依赖问题。如下所示：

![img](https://oss.xubighead.top/oss/image/202506/1930433185591693314.png)



由于把实例化和初始化的流程分开了才能解决循环依赖，所以如果都是用构造器的话，就没法分离这个操作，所以都是构造器的话就无法解决循环依赖的问题了。



#### 单例的代理对象setter注入

这种注入方式其实也比较常用，比如平时使用：`@Async`注解的场景，会通过`AOP`自动生成代理对象。

```
@Service
publicclass TestService1 {
    @Autowired
    private TestService2 testService2;
    @Async
    public void test1() {
    }
}
@Service
publicclass TestService2 {
    @Autowired
    private TestService1 testService1;
    public void test2() {
    }
}
```



程序启动会报错，出现了循环依赖：

```
org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'testService1': Bean with name 'testService1' has been injected into other beans [testService2] in its raw version as part of a circular reference, but has eventually been wrapped. This means that said other beans do not use the final version of the bean. This is often the result of over-eager type matching - consider using 'getBeanNamesOfType' with the 'allowEagerInit' flag turned off, for example.
```



![img](https://oss.xubighead.top/oss/image/202506/1930433248275566594.png)



bean初始化完成之后，后面还有一步去检查：第二级缓存 和 原始对象 是否相等。



##### 源码解析

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
	protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
			throws BeanCreationException {
		// ...
		if (earlySingletonExposure) {
			Object earlySingletonReference = getSingleton(beanName, false);
            // 二级缓存和原始对象不相等，因此抛出了异常
			if (earlySingletonReference != null) {
				if (exposedObject == bean) {
					exposedObject = earlySingletonReference;
				}
				else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
					String[] dependentBeans = getDependentBeans(beanName);
					Set<String> actualDependentBeans = new LinkedHashSet<>(dependentBeans.length);
					for (String dependentBean : dependentBeans) {
						if (!removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
							actualDependentBeans.add(dependentBean);
						}
					}
					if (!actualDependentBeans.isEmpty()) {
						throw new BeanCurrentlyInCreationException(beanName,
								"Bean with name '" + beanName + "' has been injected into other beans [" +
								StringUtils.collectionToCommaDelimitedString(actualDependentBeans) +
								"] in its raw version as part of a circular reference, but has eventually been " +
								"wrapped. This means that said other beans do not use the final version of the " +
								"bean. This is often the result of over-eager type matching - consider using " +
								"'getBeanNamesOfType' with the 'allowEagerInit' flag turned off, for example.");
					}
				}
			}
		}
		// ...
	}
```



##### 扩展

###### bean加载顺序

上述循环依赖的情况下，如果这时候把TestService1改个名字，改成：TestService6，其他的都不变。

```java
@Service
publicclass TestService6 {
    @Autowired
    private TestService2 testService2;
    @Async
    public void test1() {
    }
}
```



此时启动程序就不会出现循环依赖的异常了，因为spring是按照文件完整路径递归查找的，按路径+文件名排序，排在前面的先加载。所以TestService1比TestService2先加载，而改了文件名称之后，TestService2比TestService6先加载。

![img](https://oss.xubighead.top/oss/image/202506/1930433306739970049.png)



这种情况testService6中其实第二级缓存是空的，不需要跟原始对象判断，所以不会抛出循环依赖。



#### DependsOn循环依赖

还有一种有些特殊的场景，比如我们需要在实例化Bean A之前，先实例化Bean B，这个时候就可以使用`@DependsOn`注解。

```java
@DependsOn(value = "testService2")
@Service
publicclass TestService1 {
    @Autowired
    private TestService2 testService2;
}
@DependsOn(value = "testService1")
@Service
publicclass TestService2 {
    @Autowired
    private TestService1 testService1;
}
```



程序启动之后，执行结果：

```
Circular depends-on relationship between 'testService2' and 'testService1'
```



这个例子中本来如果TestService1和TestService2都没有加`@DependsOn`注解是没问题的，反而加了这个注解会出现循环依赖问题。

在AbstractBeanFactory类的doGetBean方法中会检查dependsOn的实例有没有循环依赖，如果有循环依赖则抛异常。



##### 源码解析

> org.springframework.beans.factory.support.AbstractBeanFactory

```java
	protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
			@Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
				// ...

				// 保证当前 bean 所依赖的 bean 的初始化。
				String[] dependsOn = mbd.getDependsOn();
				if (dependsOn != null) {
					for (String dep : dependsOn) {
						if (isDependent(beanName, dep)) {
							throw new BeanCreationException(mbd.getResourceDescription(), beanName,
									"Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
						}
						registerDependentBean(dep, beanName);
						try {
							getBean(dep);
						}
						catch (NoSuchBeanDefinitionException ex) {
							throw new BeanCreationException(mbd.getResourceDescription(), beanName,
									"'" + beanName + "' depends on missing bean '" + dep + "'", ex);
						}
					}
				}
				// ...
	}
```



### 循环依赖解决

#### 生成代理对象产生的循环依赖

这类循环依赖问题解决方法很多，主要有：

1. 使用`@Lazy`注解，延迟加载
2. 使用`@DependsOn`注解，指定加载先后关系
3. 修改文件名称，改变循环依赖类的加载顺序



#### 使用@DependsOn产生的循环依赖

这类循环依赖问题要找到`@DependsOn`注解循环依赖的地方，迫使它不循环依赖就可以解决问题。



#### 多例循环依赖

这类循环依赖问题可以通过把bean改成单例的解决。



#### 构造器循环依赖

这类循环依赖问题可以通过使用`@Lazy`注解解决



### 源码解析

#### 添加三级缓存

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
    throws BeanCreationException {
    // 实例化对象
    // ...
    // 判断是否允许提前暴露对象，如果允许，则直接添加一个 ObjectFactory 到三级缓存
    boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
                                      isSingletonCurrentlyInCreation(beanName));
    if (earlySingletonExposure) {
        // 添加三级缓存的方法详情在下方
        addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
    }
    // ...
    // 注入属性和初始化
}

/*
 * 获取对指定 bean 的早期访问的引用，通常用于解析循环引用。
 */
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
    Object exposedObject = bean;
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        for (BeanPostProcessor bp : getBeanPostProcessors()) {
            if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
                // 如果需要代理，这里会返回代理对象；否则返回原始对象
                SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
                exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);
            }
        }
    }
    return exposedObject;
}
```



这种场景下Spring定义了一个匿名内部类，通过`getEarlyBeanReference`方法获取代理对象，其实底层是通过`AbstractAutoProxyCreator`类的`getEarlyBeanReference`生成代理对象。



> org.springframework.beans.factory.support.DefaultSingletonBeanRegistry

```java
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(singletonFactory, "Singleton factory must not be null");
    synchronized (this.singletonObjects) {
        // 判断一级缓存中不存在此对象
        if (!this.singletonObjects.containsKey(beanName)) {
            // 添加至三级缓存并确保二级缓存没有该对象
            this.singletonFactories.put(beanName, singletonFactory);
            this.earlySingletonObjects.remove(beanName);
            this.registeredSingletons.add(beanName);
        }
    }
}
```



因为提前进行了代理，避免对后面重复创建代理对象，会在 `earlyProxyReferences` 中记录已被代理的对象。

> org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator

```java
@Override
public Object getEarlyBeanReference(Object bean, String beanName) {
    Object cacheKey = getCacheKey(bean.getClass(), beanName);
    this.earlyProxyReferences.put(cacheKey, bean);
    return wrapIfNecessary(bean, beanName, cacheKey);
}
```



#### 获取依赖

> org.springframework.beans.factory.support.DefaultSingletonBeanRegistry

```java
@Nullable
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    // 一级缓存
    Object singletonObject = this.singletonObjects.get(beanName);
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        synchronized (this.singletonObjects) {
            // 二级缓存
            singletonObject = this.earlySingletonObjects.get(beanName);
            if (singletonObject == null && allowEarlyReference) {
                // 三级缓存
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    // Bean 工厂中获取 Bean 后放入二级缓存并删除三级缓存
                    singletonObject = singletonFactory.getObject();
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return singletonObject;
}
```



当 Spring 为某个 Bean 填充属性的时候，它首先会寻找需要注入对象的名称，然后依次执行 `getSingleton()` 方法得到所需注入的对象，而获取对象的过程就是先从一级缓存中获取，一级缓存中没有就从二级缓存中获取，二级缓存中没有就从三级缓存中获取，如果三级缓存中也没有，那么就会去执行 `doCreateBean()` 方法创建这个 Bean。



### Summary

#### 只有二级缓存的情况

##### 保留一二级缓存

第三级缓存的目的是为了延迟代理对象的创建，因为如果没有依赖循环的话，那么就不需要为其提前创建代理，可以将它延迟到初始化完成之后再创建。

也在实例化完成之后，就为其创建代理对象，这样就不需要第三级缓存了。可以将`addSingletonFactory()` 方法进行改造：

```java
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(singletonFactory, "Singleton factory must not be null");
    synchronized (this.singletonObjects) {
        if (!this.singletonObjects.containsKey(beanName)) { // 判断一级缓存中不存在此对象
            object o = singletonFactory.getObject(); // 直接从工厂中获取 Bean
            this.earlySingletonObjects.put(beanName, o); // 添加至二级缓存中
            this.registeredSingletons.add(beanName);
        }
    }
}
```



这样的话，每次实例化完 Bean 之后就直接去创建代理对象，并添加到二级缓存中。**测试结果是完全正常的**，**Spring 的初始化时间应该也是不会有太大的影响。**

如果 Spring 选择二级缓存来解决循环依赖的话，那么就意味着所有 Bean 都需要在实例化完成之后就立马为其创建代理，而 Spring 的设计原则是在 Bean 初始化完成之后才为其创建代理。

动态代理的创建时机就是存在于初始化后，为这个bean对象生成代理对象，如果先创建的话，这样不管这个类是否存在循环依赖，都会先生成，又再一次违反Bean的初始化设计。



并不是说二级缓存如果存在aop的话就无法将代理对象注入的问题，本质应该说是初始spring是没有解决循环引用问题的，设计原则是 bean 实例化、属性设置、初始化之后 再 生成aop对象，但是为了解决循环依赖但又尽量不打破这个设计原则的情况下，使用了存储了函数式接口的第三级缓存； 如果使用二级缓存的话，可以将aop的代理工作提前到 提前暴露实例的阶段执行； 也就是说所有的bean在创建过程中就先生成代理对象再初始化和其他工作； 但是这样的话，就和spring的aop的设计原则相驳，aop的实现需要与bean的正常生命周期的创建分离； 这样只有使用第三级缓存封装一个函数式接口对象到缓存中， 发生循环依赖时，触发代理类的生成



##### 保留一三级缓存

Spring 选择了三级缓存。但是因为循环依赖的出现，导致了 Spring 不得不提前去创建代理，因为如果不提前创建代理对象，那么注入的就是原始对象，这样就会产生错误。

如果出现循环依赖+aop时，多个地方注入这个动态代理对象需要保证都是同一个对象，而三级缓存中的取出来的动态代理对象每次都是新对象，地址值不一样。从而违背单例规则。

如果只使用这两层缓存，在使用三级缓存中的工厂对象生成的动态代理对象都是新创建的，循环依赖的时候，注入到别的bean里面去的那个动态代理对象和最终这个bean在初始化后自己创建的bean地址值不一样，或者说有2个以上的bean循环依赖的时候，他们各自拿到的bean的动态代理对象都是不一样的。所以需要一个二级缓存来存，如果二级里面有就不用查三级了。这也三级缓存和二级缓存的初始容量只有16的原因出现循环依赖本身就是代码设计不合理的，不要为了那少部分的本身不合理情况的循环依赖去改变一个大多数都合理的设计。