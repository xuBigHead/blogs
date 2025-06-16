## 对象生命周期

对 Prototype Bean 来说，当用户 getBean 获得 Prototype Bean 的实例后，IOC 容器就不再对当前实例进行管理，而是把管理权交由用户，此后再 getBean 生成的是新的实例。

**所以描述 Bean 的生命周期，都是指的 Singleton Bean。**



### 实现流程

Spring Bean的生命周期有4个阶段，如下：

1. 实例化 Instantiation：第 1 步，实例化一个 Bean 对象；
2. 属性赋值 Populate：第 2 步，为 Bean 设置相关属性和依赖；
3. 初始化 Initialization：初始化的阶段的步骤比较多，5、6 步是真正的初始化，第 3、4 步为在初始化前执行，第 7 步在初始化后执行，初始化完成之后，Bean 就可以被使用了；
4. 销毁 Destruction：第 8~10 步，第 8 步其实也可以算到销毁阶段，但不是真正意义上的销毁，而是先在使用前注册了销毁的相关调用接口，为了后面第 9、10 步真正销毁 Bean 时再执行相应的方法。



![图片](https://oss.xubighead.top/oss/image/202506/1930209751515369473.jpg)



**doCreateBean()**：这个是入口；

**createBeanInstance()**：用来初始化 Bean，里面会调用对象的构造方法；

**populateBean()**：属性对象的依赖注入，以及成员变量初始化；

**initializeBean()**：里面有 4 个方法，

- 先执行 aware 的 BeanNameAware、BeanFactoryAware 接口；
- 再执行 BeanPostProcessor 前置接口；
- 然后执行 InitializingBean 接口，以及配置的 init()；
- 最后执行 BeanPostProcessor 的后置接口。

**destory()**：先执行 DisposableBean 接口，再执行配置的 destroyMethod()。



#### 入口

##### 源码解析

> org.springframework.context.support.AbstractApplicationContext

```java
@Override
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        try {
            // ...
            // 实例化剩下的普通的Bean对象
            finishBeanFactoryInitialization(beanFactory);
            // ...
        }
        // ...
    }
}

protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    // ...
    // 实例化所有剩下的非懒加载的单例对象
    beanFactory.preInstantiateSingletons();
}
```



> org.springframework.beans.factory.support.DefaultListableBeanFactory

```java
@Override
public void preInstantiateSingletons() throws BeansException {
    // ...
    for (String beanName : beanNames) {
        RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
        if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
            if (isFactoryBean(beanName)) {
                // ...
            }
            else {
                getBean(beanName);
            }
        }
    }
    // ...
}
```



> org.springframework.beans.factory.support.AbstractBeanFactory

```java
@Override
public Object getBean(String name) throws BeansException {
    return doGetBean(name, null, null, false);
}

protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
                          @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
	// ...
    if (sharedInstance != null && args == null) {
        // ...
    }
    else {
        // ...
        try {
            // ...
            if (dependsOn != null) {
                // ...
            }
            // 创建对象实例
            if (mbd.isSingleton()) {
                sharedInstance = getSingleton(beanName, () -> {
                    try {
                        // 创建对象
                        return createBean(beanName, mbd, args);
                    }
                    // ...
                });
                bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
            }
            else if (mbd.isPrototype()) {
                // ...
            }
            else {
                // ...
            }
        }
        catch (BeansException ex) {
            cleanupAfterBeanCreationFailure(beanName);
            throw ex;
        }
    }
	// ...
    return (T) bean;
}
```



> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
@Override
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
    throws BeanCreationException {
	// ...
    try {
        // Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
        Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
        if (bean != null) {
            return bean;
        }
    }
 	// ...
    try {
        Object beanInstance = doCreateBean(beanName, mbdToUse, args);
        return beanInstance;
    }
    // ...
}

protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
    throws BeanCreationException {
    // 实例化对象.
    BeanWrapper instanceWrapper = null;
    if (mbd.isSingleton()) {
        instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
    }
    if (instanceWrapper == null) {
        // 创建Bean
        instanceWrapper = createBeanInstance(beanName, mbd, args);
    }

    // 初始化对象
    Object exposedObject = bean;
    try {
        // 属性赋值
        populateBean(beanName, mbd, instanceWrapper);
        // 初始化
        exposedObject = initializeBean(beanName, exposedObject, mbd);
    }
    catch (Throwable ex) {
        // ...
    }
    // ...
    return exposedObject;
}
```



#### 实例化

实例化，创建一个Bean对象。

Bean实例化的时机也分为两种，BeanFactory管理的Bean是在使用到Bean的时候才会实例化Bean，ApplicantContext管理的Bean在容器初始化的时候就回完成Bean实例化。



##### 源码解析

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
	// ...
    // 无特殊处理的空构造器的对象
    return instantiateBean(beanName, mbd);
}

protected BeanWrapper instantiateBean(final String beanName, final RootBeanDefinition mbd) {
    try {
        Object beanInstance;
        final BeanFactory parent = this;
        if (System.getSecurityManager() != null) {
            // ...
        }
        else {
            beanInstance = getInstantiationStrategy().instantiate(mbd, beanName, parent);
        }
        BeanWrapper bw = new BeanWrapperImpl(beanInstance);
        initBeanWrapper(bw);
        return bw;
    }
	// ...
}
```



> org.springframework.beans.factory.support.SimpleInstantiationStrategy

```java
@Override
public Object instantiate(RootBeanDefinition bd, @Nullable String beanName, BeanFactory owner) {
    if (!bd.hasMethodOverrides()) {
        Constructor<?> constructorToUse;
        synchronized (bd.constructorArgumentLock) {
            // ...
        }
        // 构造器创建对象
        return BeanUtils.instantiateClass(constructorToUse);
    }
    else {
        return instantiateWithMethodInjection(bd, beanName, owner);
    }
}
```



#### 属性赋值

填充属性，为属性赋值。



##### 源码解析

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
    // ...
    if (pvs != null) {
        applyPropertyValues(beanName, mbd, bw, pvs);
    }
}

protected void applyPropertyValues(String beanName, BeanDefinition mbd, BeanWrapper bw, PropertyValues pvs) {
    // 设置属性
    try {
        bw.setPropertyValues(new MutablePropertyValues(deepCopy));
    }
	// ...
}
```



> org.springframework.beans.AbstractPropertyAccessor

```java
@Override
public void setPropertyValues(PropertyValues pvs) throws BeansException {
    setPropertyValues(pvs, false, false);
}

@Override
public void setPropertyValues(PropertyValues pvs, boolean ignoreUnknown, boolean ignoreInvalid)
    throws BeansException {
    // ...
    for (PropertyValue pv : propertyValues) {
        try {
          	// 填充属性
            setPropertyValue(pv);
        }
        // ...
    }
	// ...
}
```



> org.springframework.beans.AbstractNestablePropertyAccessor

```java
@Override
public void setPropertyValue(PropertyValue pv) throws BeansException {
    PropertyTokenHolder tokens = (PropertyTokenHolder) pv.resolvedTokens;
    if (tokens == null) {
        // ...
    }
    else {
        setPropertyValue(tokens, pv);
    }
}

protected void setPropertyValue(PropertyTokenHolder tokens, PropertyValue pv) throws BeansException {
    if (tokens.keys != null) {
        processKeyedProperty(tokens, pv);
    }
    else {
        processLocalProperty(tokens, pv);
    }
}

private void processLocalProperty(PropertyTokenHolder tokens, PropertyValue pv) {
    PropertyHandler ph = getLocalPropertyHandler(tokens.actualName);
	// ...

    Object oldValue = null;
    try {
        Object originalValue = pv.getValue();
        Object valueToApply = originalValue;
        // ...
        ph.setValue(valueToApply);
    }
	// ...
}
```



> org.springframework.beans.BeanWrapperImpl.BeanPropertyHandler

```java
@Override
public void setValue(final @Nullable Object value) throws Exception {
    final Method writeMethod = (this.pd instanceof GenericTypeAwarePropertyDescriptor ?
                                ((GenericTypeAwarePropertyDescriptor) this.pd).getWriteMethodForActualAccess() :
                                this.pd.getWriteMethod());
    if (System.getSecurityManager() != null) {
       	// ...
    }
    else {
        ReflectionUtils.makeAccessible(writeMethod);
        writeMethod.invoke(getWrappedInstance(), value);
    }
}
```



#### 初始化

- 如果实现了`xxxAware`接口，通过不同类型的Aware接口拿到Spring容器的资源
- 如果实现了BeanPostProcessor接口，则会回调该接口的`postProcessBeforeInitialzation`和`postProcessAfterInitialization`方法
- 如果配置了`init-method`方法，则会执行`init-method`配置的方法



##### 初始化方式

因为这几个方法对应的都是同一个生命周期，只是实现方式不同，我们一般只采用其中一种方式。这几种定义方式的调用顺序如下：

**Constructor > @PostConstruct > InitializingBean > init-method**



###### @PostConstruct注解

使用@PostConstruct注解，该注解作用于void方法上。触发点是在`postProcessBeforeInitialization`之后，`InitializingBean.afterPropertiesSet`之前。



```java
public class NormalBeanA {
    public NormalBeanA() {
        System.out.println("NormalBean constructor");
    }

    @PostConstruct
    public void init(){
        System.out.println("[PostConstruct] NormalBeanA");
    }
}
```



###### 配置init-method

在配置文件中配置init-method方法



###### 实现InitializingBean接口

将类实现InitializingBean接口，`InitializingBean`接口为bean提供了初始化方法的方式，它只包括`afterPropertiesSet`方法，凡是继承该接口的类，在初始化bean的时候都会执行该方法。这个扩展点的触发时机在`postProcessAfterInitialization`之前。

实现此接口，来进行系统启动的时候一些业务指标的初始化工作。



##### 源码解析

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
    if (System.getSecurityManager() != null) {
        // ...
    }
    else {
        // 调用Aware接口相关方法
        invokeAwareMethods(beanName, bean);
    }

    Object wrappedBean = bean;
    if (mbd == null || !mbd.isSynthetic()) {
       	// 调用BeanPostProcesser接口
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
    }

    try {
        // 初始化方法
        invokeInitMethods(beanName, wrappedBean, mbd);
    }
    // ...
    if (mbd == null || !mbd.isSynthetic()) {
        // 调用BeanPostProcesser接口
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    }

    return wrappedBean;
}

protected void invokeInitMethods(String beanName, final Object bean, @Nullable RootBeanDefinition mbd)
    throws Throwable {

    boolean isInitializingBean = (bean instanceof InitializingBean);
    if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
        // ...
        if (System.getSecurityManager() != null) {
            // ...
        }
        else {
            // 调用实现InitializingBean接口的方法
            ((InitializingBean) bean).afterPropertiesSet();
        }
    }

    if (mbd != null && bean.getClass() != NullBean.class) {
        String initMethodName = mbd.getInitMethodName();
        if (StringUtils.hasLength(initMethodName) &&
            !(isInitializingBean && "afterPropertiesSet".equals(initMethodName)) &&
            !mbd.isExternallyManagedInitMethod(initMethodName)) {
            // 调用自定义初始化方法
            invokeCustomInitMethod(beanName, bean, mbd);
        }
    }
}
```



#### 销毁

- 容器关闭后，如果Bean实现了`DisposableBean`接口，则会回调该接口的`destroy`方法
- 如果配置了`destroy-method`方法，则会执行`destroy-method`配置的方法



##### 销毁方式

###### @PreDestroy注解



###### 配置destroy-method



###### 实现了DisposableBean接口



##### 源码解析

销毁，是在容器关闭时调用的，详见 ConfigurableApplicationContext#close()

> org.springframework.context.ConfigurableApplicationContext

```java
	/**
	 * 
	 */
	@Override
	void close();
```



ConfigurableApplicationContext接口定义了close方法，由AbstractApplicationContext实现。



> org.springframework.context.support.AbstractApplicationContext

```java
@Override
public void close() {
    synchronized (this.startupShutdownMonitor) {
        doClose();
        // ...
    }
}

protected void doClose() {
    // 检查实际关闭尝试是否必要
    if (this.active.get() && this.closed.compareAndSet(false, true)) {
        // ...
        // 销毁所有缓存的单例对象
        destroyBeans();
        // ...
    }
}

protected void destroyBeans() {
    getBeanFactory().destroySingletons();
}
```



> org.springframework.beans.factory.support.DefaultListableBeanFactory

```java
@Override
public void destroySingletons() {
    super.destroySingletons();
    updateManualSingletonNames(Set::clear, set -> !set.isEmpty());
    clearByTypeCache();
}
```



> org.springframework.beans.factory.support.DefaultSingletonBeanRegistry

```java
public void destroySingletons() {
	// ...
    String[] disposableBeanNames;
    synchronized (this.disposableBeans) {
        disposableBeanNames = StringUtils.toStringArray(this.disposableBeans.keySet());
    }
    for (int i = disposableBeanNames.length - 1; i >= 0; i--) {
        destroySingleton(disposableBeanNames[i]);
    }
	// ...
}
```



> org.springframework.beans.factory.support.DefaultListableBeanFactory

```java
@Override
public void destroySingleton(String beanName) {
    super.destroySingleton(beanName);
    removeManualSingletonName(beanName);
    clearByTypeCache();
}
```



> org.springframework.beans.factory.support.DefaultSingletonBeanRegistry

```java
public void destroySingleton(String beanName) {
    // Remove a registered singleton of the given name, if any.
    removeSingleton(beanName);

    // Destroy the corresponding DisposableBean instance.
    DisposableBean disposableBean;
    synchronized (this.disposableBeans) {
        disposableBean = (DisposableBean) this.disposableBeans.remove(beanName);
    }
    destroyBean(beanName, disposableBean);
}

protected void destroyBean(String beanName, @Nullable DisposableBean bean) {
    // Trigger destruction of dependent beans first...
    Set<String> dependencies;
    synchronized (this.dependentBeanMap) {
        // Within full synchronization in order to guarantee a disconnected Set
        dependencies = this.dependentBeanMap.remove(beanName);
    }
    if (dependencies != null) {
        // ...
        for (String dependentBeanName : dependencies) {
            destroySingleton(dependentBeanName);
        }
    }

    // Actually destroy the bean now...
    if (bean != null) {
        try {
            // 调用DispoableBean接口实现的方法销毁对象
            bean.destroy();
        }
		// ...
    }

    // Trigger destruction of contained beans...
    Set<String> containedBeans;
    synchronized (this.containedBeanMap) {
        // Within full synchronization in order to guarantee a disconnected Set
        containedBeans = this.containedBeanMap.remove(beanName);
    }
    if (containedBeans != null) {
        for (String containedBeanName : containedBeans) {
            destroySingleton(containedBeanName);
        }
    }

    // Remove destroyed bean from other beans' dependencies.
    synchronized (this.dependentBeanMap) {
        for (Iterator<Map.Entry<String, Set<String>>> it = 
             this.dependentBeanMap.entrySet().iterator(); it.hasNext();) {
            Map.Entry<String, Set<String>> entry = it.next();
            Set<String> dependenciesToClean = entry.getValue();
            dependenciesToClean.remove(beanName);
            if (dependenciesToClean.isEmpty()) {
                it.remove();
            }
        }
    }

    // Remove destroyed bean's prepared dependency information.
    this.dependenciesForBeanMap.remove(beanName);
}
```



### 扩展点

Aware 接口：让 Bean 能拿到容器的一些资源，例如 BeanNameAware 的 **setBeanName()**，BeanFactoryAware 的 **setBeanFactory()**；

后处理器：进行一些前置和后置的处理，例如 BeanPostProcessor 的 **postProcessBeforeInitialization()** 和 **postProcessAfterInitialization()**；

生命周期接口：定义初始化方法和销毁方法的，例如 InitializingBean 的 **afterPropertiesSet()**，以及 DisposableBean 的 **destroy()**；

配置生命周期方法：可以通过配置文件，自定义初始化和销毁方法，例如配置文件配置的 **init()** 和 **destroyMethod()**。



InstantiationAwareBeanPostProcessor作用于**实例化**阶段的前后，BeanPostProcessor作用于**初始化**阶段的前后。

![img](https://oss.xubighead.top/oss/image/202506/1930209875566104577.png)

#### InstantiationAwareBeanPostProcessor

```java
public interface InstantiationAwareBeanPostProcessor extends BeanPostProcessor {
	/**
	 * 
	 */
	@Nullable
	default Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
		return null;
	}

	/**
	 * 
	 */
	default boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
		return true;
	}
}
```



##### 调用时机

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
    throws BeanCreationException {
    try {
        // 让 BeanPostProcessors 有机会返回一个代理而不是目标 bean 实例。
        Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
        if (bean != null) {
            return bean;
        }
    }
    catch (Throwable ex) {
        // ...
    }

    try {
        // 创建对象
        Object beanInstance = doCreateBean(beanName, mbdToUse, args);
        // ...
        return beanInstance;
    }
    catch (BeanCreationException | ImplicitlyAppearedSingletonException ex) {
        // ...
    }
}
```



> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
    Object bean = null;
    if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
        // Make sure bean class is actually resolved at this point.
        if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
            Class<?> targetType = determineTargetType(beanName, mbd);
            if (targetType != null) {
                bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
                if (bean != null) {
                    bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
                }
            }
        }
        mbd.beforeInstantiationResolved = (bean != null);
    }
    return bean;
}
```



postProcessBeforeInstantiation在doCreateBean之前调用，也就是在bean实例化之前调用的，英文源码注释解释道该方法的返回值会替换原本的Bean作为代理，这也是Aop等功能实现的关键点。



###### postProcessBeforeInstantiation

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
@Nullable
protected Object applyBeanPostProcessorsBeforeInstantiation(Class<?> beanClass, String beanName) {
    for (BeanPostProcessor bp : getBeanPostProcessors()) {
        if (bp instanceof InstantiationAwareBeanPostProcessor) {
            InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
            Object result = ibp.postProcessBeforeInstantiation(beanClass, beanName);
            if (result != null) {
                return result;
            }
        }
    }
    return null;
}
```



###### postProcessAfterInstantiation

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
    // ...
    // 让 InstantiationAwareBeanPostProcessors 在设置属性之前修改 bean 的状态。
    // 例如，这可以用于支持字段注入的样式。
    // 方法作为属性赋值的前置检查条件，在属性赋值之前执行，能够影响是否进行属性赋值！
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        for (BeanPostProcessor bp : getBeanPostProcessors()) {
            if (bp instanceof InstantiationAwareBeanPostProcessor) {
                InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
                if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
                    return;
                }
            }
        }
    }
    // ...
}
```



#### Aware接口

Aware接口的作用是让Bean拿到Spring容器中的资源**所有的Aware方法都是在初始化阶段之前调用的！**

Aware之前的名字就是可以拿到什么资源，例如BeanNameAware可以拿到BeanName，以此类推。调用时机需要注意：



`Aware` 是一个空接口，里面不包括任何方法。该接口表示已感知，可以通过该接口的实现获取指定对象。

> org.springframework.beans.factory.Aware

```java
public interface Aware {
}
```



Aware接口具体可以分为两组，如下所示：



- 对象相关：

1. BeanNameAware
2. BeanClassLoaderAware
3. BeanFactoryAware



- ApplicationContext相关：

1. EnvironmentAware
2. EmbeddedValueResolverAware 实现该接口能够获取Spring EL解析器，用户的自定义注解需要支持spel表达式的时候可以使用，非常方便。
3. ApplicationContextAware(ResourceLoaderAware\ApplicationEventPublisherAware\MessageSourceAware) 这几个接口可能让人有点懵，实际上这几个接口可以一起记，其返回值实质上都是当前的ApplicationContext对象，因为ApplicationContext是一个复合接口。



该接口的部分实现如下：

- ApplicationEventPublisherAware
- ServletContextAware
- MessageSourceAware
- ResourceLoaderAware
- SchedulerContextAware
- NotificationPublisherAware
- EnvironmentAware
- BeanFactoryAware
- EmbeddedValueResolverAware
- ImportAware
- ServletConfigAware
- BootstrapContextAware
- LoadTimeWeaverAware
- BeanNameAware
- BeanClassLoaderAware
- ApplicationContextAware



并不是所有的Aware接口都使用同样的方式调用。Bean××Aware都是在代码中直接调用的，而ApplicationContext相关的Aware都是通过BeanPostProcessor#postProcessBeforeInitialization()实现的。



##### 调用时机

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
	protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
		if (System.getSecurityManager() != null) {
			AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
                // 调用对象相关的Aware
				invokeAwareMethods(beanName, bean);
				return null;
			}, getAccessControlContext());
		}
		else {
            // 调用对象相关的Aware
			invokeAwareMethods(beanName, bean);
		}
		// ...
	}
```



###### 对象相关

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
	private void invokeAwareMethods(final String beanName, final Object bean) {
		if (bean instanceof Aware) {
			if (bean instanceof BeanNameAware) {
				((BeanNameAware) bean).setBeanName(beanName);
			}
			if (bean instanceof BeanClassLoaderAware) {
				ClassLoader bcl = getBeanClassLoader();
				if (bcl != null) {
					((BeanClassLoaderAware) bean).setBeanClassLoader(bcl);
				}
			}
			if (bean instanceof BeanFactoryAware) {
				((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
			}
		}
	}
```



###### ApplicationContext相关

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
	@Override
	public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
			throws BeansException {

		Object result = existingBean;
		for (BeanPostProcessor processor : getBeanPostProcessors()) {
			Object current = processor.postProcessBeforeInitialization(result, beanName);
			if (current == null) {
				return result;
			}
			result = current;
		}
		return result;
	}

```



> org.springframework.context.support.ApplicationContextAwareProcessor

```java
	public Object postProcessBeforeInitialization(final Object bean, String beanName) throws BeansException {
		AccessControlContext acc = null;

		if (System.getSecurityManager() != null &&
				(bean instanceof EnvironmentAware || bean instanceof EmbeddedValueResolverAware ||
						bean instanceof ResourceLoaderAware || bean instanceof ApplicationEventPublisherAware ||
						bean instanceof MessageSourceAware || bean instanceof ApplicationContextAware)) {
			acc = this.applicationContext.getBeanFactory().getAccessControlContext();
		}

		if (acc != null) {
			AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
				invokeAwareInterfaces(bean);
				return null;
			}, acc);
		}
		else {
			invokeAwareInterfaces(bean);
		}

		return bean;
	}

	private void invokeAwareInterfaces(Object bean) {
		if (bean instanceof Aware) {
			if (bean instanceof EnvironmentAware) {
				((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
			}
			if (bean instanceof EmbeddedValueResolverAware) {
				((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
			}
			if (bean instanceof ResourceLoaderAware) {
				((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
			}
			if (bean instanceof ApplicationEventPublisherAware) {
				((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
			}
			if (bean instanceof MessageSourceAware) {
				((MessageSourceAware) bean).setMessageSource(this.applicationContext);
			}
			if (bean instanceof ApplicationContextAware) {
				((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
			}
		}
	}
```

ApplicationContext相关的Aware是通过BeanPostProcessor（ApplicationContextAwareProcessor）实现的。



#### BeanPostProcessor

从refresh方法来看,BeanPostProcessor 实例化比正常的bean早，从initializeBean方法看,每个bean初始化前后都调用所有BeanPostProcessor的postProcessBeforeInitialization和postProcessAfterInitialization方法。

> org.springframework.beans.factory.config.BeanPostProcessor

```java
public interface BeanPostProcessor {
	/**
	 * 
	 */
	@Nullable
	default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

	/**
	 * 
	 */
	@Nullable
	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}
}
```



##### 注册时机

BeanPostProcessor会在业务Bean之前初始化完成。



> org.springframework.context.support.AbstractApplicationContext

```java
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // ...
        try {
            // ...
            // 注册拦截bean创建的BeanPostProcessor
            registerBeanPostProcessors(beanFactory);

            // ...

            // 实例化并初始化bean
            finishBeanFactoryInitialization(beanFactory);
            // ...
        }
        // ...
    }
}
```



Spring是先执行registerBeanPostProcessors()进行BeanPostProcessors的注册，然后再执行finishBeanFactoryInitialization创建我们的单例非懒加载的Bean。



##### 执行顺序

BeanPostProcessor有很多个，而且每个BeanPostProcessor都影响多个Bean，其执行顺序至关重要，必须能够控制其执行顺序才行。关于执行顺序这里需要引入两个排序相关的接口：PriorityOrdered、Ordered

- PriorityOrdered是一等公民，首先被执行，PriorityOrdered公民之间通过接口返回值排序
- Ordered是二等公民，然后执行，Ordered公民之间通过接口返回值排序
- 都没有实现是三等公民，最后执行



> org.springframework.context.support.AbstractApplicationContext

```java
	protected void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory) {
		PostProcessorRegistrationDelegate.registerBeanPostProcessors(beanFactory, this);
	}
```



> org.springframework.context.support.PostProcessorRegistrationDelegate

```java
	public static void registerBeanPostProcessors(
			ConfigurableListableBeanFactory beanFactory, AbstractApplicationContext applicationContext) {

		String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);

		// Register BeanPostProcessorChecker that logs an info message when
		// a bean is created during BeanPostProcessor instantiation, i.e. when
		// a bean is not eligible for getting processed by all BeanPostProcessors.
		int beanProcessorTargetCount = beanFactory.getBeanPostProcessorCount() + 1 + postProcessorNames.length;
		beanFactory.addBeanPostProcessor(new BeanPostProcessorChecker(beanFactory, beanProcessorTargetCount));

		// Separate between BeanPostProcessors that implement PriorityOrdered,
		// Ordered, and the rest.
		List<BeanPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
		List<BeanPostProcessor> internalPostProcessors = new ArrayList<>();
		List<String> orderedPostProcessorNames = new ArrayList<>();
		List<String> nonOrderedPostProcessorNames = new ArrayList<>();
		for (String ppName : postProcessorNames) {
			if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
				BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
				priorityOrderedPostProcessors.add(pp);
				if (pp instanceof MergedBeanDefinitionPostProcessor) {
					internalPostProcessors.add(pp);
				}
			}
			else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
				orderedPostProcessorNames.add(ppName);
			}
			else {
				nonOrderedPostProcessorNames.add(ppName);
			}
		}

		// 首先注册实现了PriorityOrdered的
		sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
		registerBeanPostProcessors(beanFactory, priorityOrderedPostProcessors);

		// 然后注册实现了Ordered的BeanPostProcessor并排序
		List<BeanPostProcessor> orderedPostProcessors = new ArrayList<>();
		for (String ppName : orderedPostProcessorNames) {
			BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
			orderedPostProcessors.add(pp);
			if (pp instanceof MergedBeanDefinitionPostProcessor) {
				internalPostProcessors.add(pp);
			}
		}
		sortPostProcessors(orderedPostProcessors, beanFactory);
		registerBeanPostProcessors(beanFactory, orderedPostProcessors);

		// 最后注册其它BeanPostProcessor
		List<BeanPostProcessor> nonOrderedPostProcessors = new ArrayList<>();
		for (String ppName : nonOrderedPostProcessorNames) {
			BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
			nonOrderedPostProcessors.add(pp);
			if (pp instanceof MergedBeanDefinitionPostProcessor) {
				internalPostProcessors.add(pp);
			}
		}
		registerBeanPostProcessors(beanFactory, nonOrderedPostProcessors);

		// Finally, re-register all internal BeanPostProcessors.
		sortPostProcessors(internalPostProcessors, beanFactory);
		registerBeanPostProcessors(beanFactory, internalPostProcessors);

		// Re-register post-processor for detecting inner beans as ApplicationListeners,
		// moving it to the end of the processor chain (for picking up proxies etc).
		beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(applicationContext));
	}
```



###### PriorityOrdered

> org.springframework.core.PriorityOrdered

```java
public interface PriorityOrdered extends Ordered {
}
```



###### Ordered

> org.springframework.core.Ordered

```java
public interface Ordered {
	/**
	 * 
	 */
	int HIGHEST_PRECEDENCE = Integer.MIN_VALUE;

	/**
	 * 
	 */
	int LOWEST_PRECEDENCE = Integer.MAX_VALUE;

	/**
	 * 
	 */
	int getOrder();
}
```

根据排序接口返回值排序，默认升序排序，返回值越低优先级越高。

PriorityOrdered、Ordered接口作为Spring整个框架通用的排序接口，在Spring中应用广泛，也是非常重要的接口。



##### 调用时机

###### postProcessBeforeInitialization

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
    // ...
    Object wrappedBean = bean;
    if (mbd == null || !mbd.isSynthetic()) {
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
    }
    // ...
    return wrappedBean;
}
```



> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
    throws BeansException {

    Object result = existingBean;
    for (BeanPostProcessor processor : getBeanPostProcessors()) {
        Object current = processor.postProcessBeforeInitialization(result, beanName);
        if (current == null) {
            return result;
        }
        result = current;
    }
    return result;
}
```



###### postProcessAfterInitialization

- 场景一

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
	// ...
                if (bean != null) {
                    bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
	// ...
}
```



> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
	public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
			throws BeansException {

		Object result = existingBean;
		for (BeanPostProcessor processor : getBeanPostProcessors()) {
			Object current = processor.postProcessAfterInitialization(result, beanName);
			if (current == null) {
				return result;
			}
			result = current;
		}
		return result;
	}
```



- 场景二

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
	protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
		// ...
		if (mbd == null || !mbd.isSynthetic()) {
			wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
		}

		return wrappedBean;
	}
```



#### InitializingBean

InitializingBean顾名思义，是初始化Bean相关的接口。



> org.springframework.beans.factory.InitializingBean

```java
public interface InitializingBean {
	/**
	 * 
	 */
	void afterPropertiesSet() throws Exception;

}
```



##### 调用时机

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
	protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
		try {
			invokeInitMethods(beanName, wrappedBean, mbd);
		}
		// ...
		return wrappedBean;
	}
```



> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
	protected void invokeInitMethods(String beanName, final Object bean, @Nullable RootBeanDefinition mbd)
			throws Throwable {

		boolean isInitializingBean = (bean instanceof InitializingBean);
		if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
			// ...
			if (System.getSecurityManager() != null) {
				// ...
			}
			else {
				((InitializingBean) bean).afterPropertiesSet();
			}
		}
		// ...
	}
```



看方法名，是在读完Properties文件，之后执行的方法。afterPropertiesSet()方法是在初始化过程中被调用的。

InitializingBean 对应生命周期的初始化阶段，在上面源码的invokeInitMethods(beanName, wrappedBean, mbd);方法中调用。

有一点需要注意，因为Aware方法都是执行在初始化方法之前，所以可以在初始化方法中放心大胆的使用Aware接口获取的资源，这也是我们自定义扩展Spring的常用方式。



#### SmartInitializingSingleton

这个接口中只有一个方法`afterSingletonsInstantiated`，其作用是是 在spring容器管理的所有单例对象（非懒加载对象）初始化完成之后调用的回调接口。其触发时机为`postProcessAfterInitialization`之后。

可以扩展此接口在对所有单例对象初始化完毕后，做一些后置的业务处理。



> org.springframework.beans.factory.SmartInitializingSingleton

```java
public interface SmartInitializingSingleton {
	void afterSingletonsInstantiated();
}
```



#### DisposableBean

DisposableBean 类似于InitializingBean，对应生命周期的销毁阶段，**以ConfigurableApplicationContext#close()方法作为入口**，实现是通过循环获取所有实现了DisposableBean接口的Bean然后调用其destroy()方法 。



> org.springframework.beans.factory.DisposableBean

```java
public interface DisposableBean {
	/**
	 * 
	 */
	void destroy() throws Exception;
}
```



也就是说，在对象销毁的时候，会去调用DisposableBean的destroy方法。在进入到销毁过程时先去调用一下DisposableBean的destroy方法，然后后执行 destroy-method声明的方法（用来销毁Bean中的各项数据）。



> org.springframework.beans.factory.support.DefaultSingletonBeanRegistry

```java
protected void destroyBean(String beanName, @Nullable DisposableBean bean) {
    // ...
    // 实际销毁对象
    if (bean != null) {
        try {
            bean.destroy();
        }
        // ...
    }
    // ...
}
```



### 总结

![img](https://oss.xubighead.top/oss/image/202506/1930209970839719937.png)



Spring Bean的生命周期分为`四个阶段`和`多个扩展点`。扩展点又可以分为`影响多个Bean`和`影响单个Bean`。整理如下：



#### 四个阶段

**实例化 > 属性赋值 > 初始化 > 销毁**



#### 多个扩展点

- 影响多个Bean
    - BeanPostProcessor
    - InstantiationAwareBeanPostProcessor
- 影响单个Bean
    - Aware
        - 对象相关Aware
            - BeanNameAware
            - BeanClassLoaderAware
            - BeanFactoryAware
        - ApplicationContext相关Aware
            - EnvironmentAware
            - EmbeddedValueResolverAware
            - ApplicationContextAware(ResourceLoaderAware\ApplicationEventPublisherAware\MessageSourceAware)
    - 生命周期
        - InitializingBean
        - DisposableBean
