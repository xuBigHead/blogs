---
layout: post
title: 第011章-Spring AOP
categories: [Spring]
description: 
keywords: Spring AOP.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# Spring_AOP

## 概念

OOP(Object-Oriented Programming)面向对象编程，允许开发者定义纵向的关系，但并适用于定义横向的关系，导致了大量代码的重复，而不利于各个模块的重用。

AOP(Aspect-Oriented Programming)，一般称为面向切面编程，作为面向对象的一种补充，用于将那些与业务无关，但却对多个对象产生影响的公共行为和逻辑，抽取并封装为一个可重用的模块，这个模块被命名为“切面”（Aspect），减少系统中的重复代码，降低了模块间的耦合度，同时提高了系统的可维护性。可用于权限认证、日志、事务处理等。



AOP(Aspect-Oriented Programming:面向切面编程)能够将那些与业务无关，**却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来**，便于**减少系统的重复代码**，**降低模块间的耦合度**，并**有利于未来的可拓展性和可维护性**。

**Spring AOP就是基于动态代理的**，如果要代理的对象，实现了某个接口，那么Spring AOP会使用**JDK Proxy**，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候Spring AOP会使用**Cglib** ，这时候Spring AOP会使用 **Cglib** 生成一个被代理对象的子类来作为代理，如下图所示：

![SpringAOPProcess](https://oss.xubighead.top/oss/image/202506/1930082388681003009.png)



通过在代理类中包裹切面，Spring在运行期把切面织入到Spring管理的bean中。代理封装了目标类，并拦截被通知方法的调用，再把调用转发给真正的目标bean。当代理拦截到方法调用时，在调用目标bean方法之前，会执行切面逻辑。

直到应用需要被代理的bean时，Spring才创建代理对象。如果使用的是ApplicationContext的话，在ApplicationContext从BeanFactory中加载所有bean的时候，Spring才会创建被代理的对象。因为Spring运行时才创建代理对象，所以我们不需要特殊的编译器来织入SpringAOP的切面。



AOP 的全称是 “Aspect Oriented Programming”，即**面向切面编程**。

在 AOP 的思想里面，周边功能（比如性能统计，日志，事务管理等）被定义为**切面**，核心功能和切面功能分别独立进行开发，然后**把核心功能和切面功能“编织”在一起，这就叫 AOP。**

AOP 能够将那些与业务无关，却为业务模块所共同调用的逻辑封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性。



### 名词概述

- **连接点(Join point)**：能够被拦截的地方，Spring AOP 是基于动态代理的，所以是方法拦截的，每个成员方法都可以称之为连接点；
- **切点(Poincut)**：每个方法都可以称之为连接点，我们具体定位到某一个方法就成为切点；
- **增强/通知(Advice)**：表示添加到切点的一段逻辑代码，并定位连接点的方位信息，简单来说就定义了是干什么的，具体是在哪干；
- **织入(Weaving)**：将增强/通知添加到目标类的具体连接点上的过程；
- **引入/引介(Introduction)**：允许我们向现有的类添加新方法或属性，是一种特殊的增强；
- **切面(Aspect)**：切面由切点和增强/通知组成，它既包括了横切逻辑的定义、也包括了连接点的定义。

上面的解释偏官方，下面用“方言”再给大家解释一遍。

- 切入点（Pointcut）：在哪些类，哪些方法上切入（**where**）；
- 通知（Advice）：在方法执行的什么时机（**when**：方法前/方法后/方法前后）做什么（**what**：增强的功能）；
- 切面（Aspect）：切面 = 切入点 + 通知，通俗点就是在什么时机，什么地方，做什么增强；
- 织入（Weaving）：把切面加入到对象，并创建出代理对象的过程，这个由 Spring 来完成。



#### 切面

（1）切面（Aspect）：切面是通知和切点的结合。通知和切点共同定义了切面的全部内容。 在Spring AOP中，切面可以使用通用类（基于模式的风格） 或者在普通类中以 @AspectJ 注解来实现。

它既包含了横切逻辑的定义, 也包括了连接点的定义. Spring AOP 就是负责实施切面的框架, 它将切面所定义的横切逻辑编织到切面所指定的连接点中.
AOP 的工作重心在于如何将增强编织目标对象的连接点上, 这里包含两个工作:

- 如何通过 pointcut 和 advice 定位到特定的 joinpoint 上
- 如何在 advice 中编写切面代码.

可以简单地认为, 使用 @Aspect 注解的类就是切面。



#### 连接点

（2）连接点（Join point）：指方法，在Spring AOP中，一个连接点 总是 代表一个方法的执行。 应用可能有数以千计的时机应用通知。这些时机被称为连接点。连接点是在应用执行过程中能够插入切面的一个点。这个点可以是调用方法时、抛出异常时、甚至修改一个字段时。切面代码可以利用这些点插入到应用的正常流程之中，并添加新的行为。

**因为Spring基于动态代理，所以Spring只支持方法连接点。**Spring缺少对字段连接点的支持，而且它不支持构造器连接点。方法之外的连接点拦截功能，我们可以利用Aspect来补充。



#### 通知

（3）通知（Advice）：在AOP术语中，切面的工作被称为通知，实际上是程序执行时要通过SpringAOP框架触发的代码段。



##### 通知类型

Spring切面可以应用5种类型的通知：

![img](https://oss.xubighead.top/oss/image/202506/1930089205737820161.png)



1. 前置通知（Before）：在目标方法被调用之前调用通知功能；
2. 后置通知（After）：在目标方法完成之后调用通知，此时不会关心方法的输出是什么；
3. 返回通知（After-returning ）：在目标方法成功执行之后调用通知；
4. 异常通知（After-throwing）：在目标方法抛出异常后调用通知；
5. 环绕通知（Around）：通知包裹了被通知的方法，在被通知的方法调用之前和调用之后执行自定义的行为。



#### 切入点

（4）切入点（Pointcut）：切点的定义会匹配通知所要织入的一个或多个连接点。我们通常使用明确的类和方法名称，或是利用正则表达式定义所匹配的类和方法名称来指定这些切点。



#### 引入

（5）引入（Introduction）：引入允许我们向现有类添加新方法或属性。



#### 目标对象

（6）目标对象（Target Object）： 被一个或者多个切面（aspect）所通知（advise）的对象。它通常是一个代理对象。也有人把它叫做 被通知（adviced） 对象。 既然Spring AOP是通过运行时代理实现的，这个对象永远是一个 被代理（proxied） 对象。



#### 织入

（7）织入（Weaving）：织入是把切面应用到目标对象并创建新的代理对象的过程。在目标对象的生命周期里有多少个点可以进行织入：

- 编译期：切面在目标类编译时被织入。AspectJ的织入编译器是以这种方式织入切面的。
- 类加载期：切面在目标类加载到JVM时被织入。需要特殊的类加载器，它可以在目标类被引入应用之前增强该目标类的字节码。AspectJ5的加载时织入就支持以这种方式织入切面。
- 运行期：切面在应用运行的某个时刻被织入。一般情况下，在织入切面时，AOP容器会为目标对象动态地创建一个代理对象。SpringAOP就是以这种方式织入切面。



### 扩展

#### Spring AOP 和 AspectJ AOP 有什么区别？

**Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。** Spring AOP 基于代理(Proxying)，而 AspectJ 基于字节码操作(Bytecode Manipulation)。

Spring AOP 已经集成了 AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。AspectJ 相比于 Spring AOP 功能更加强大，但是 Spring AOP 相对来说更简单，

如果我们的切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择 AspectJ ，它比Spring AOP 快很多。



AOP实现的关键在于 代理模式，AOP代理主要分为静态代理和动态代理。静态代理的代表为AspectJ；动态代理则以Spring AOP为代表。

静态代理与动态代理区别在于生成AOP代理对象的时机不同，相对来说AspectJ的静态代理方式具有更好的性能，但是AspectJ需要特定的编译器进行处理，而Spring AOP则无需特定的编译器处理。



（1）AspectJ是静态代理的增强，所谓静态代理，就是AOP框架会在编译阶段生成AOP代理类，因此也称为编译时增强，他会在编译阶段将AspectJ(切面)织入到Java字节码中，运行的时候就是增强之后的AOP对象。

（2）Spring AOP使用的动态代理，所谓的动态代理就是说AOP框架不会去修改字节码，而是每次运行时在内存中临时为方法生成一个AOP对象，这个AOP对象包含了目标对象的全部方法，并且在特定的切点做了增强处理，并回调原对象的方法。



#### JDK动态代理和CGLIB动态代理的区别

Spring AOP中的动态代理主要有两种方式，JDK动态代理和CGLIB动态代理：

JDK动态代理只提供接口的代理，不支持类的代理。核心InvocationHandler接口和Proxy类，InvocationHandler 通过invoke()方法反射来调用目标类中的代码，动态地将横切逻辑和业务编织在一起；接着，Proxy利用 InvocationHandler动态创建一个符合某一接口的的实例, 生成目标类的代理对象。

如果代理类没有实现接口，那么Spring AOP会选择使用CGLIB来动态代理目标类。CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成指定类的一个子类对象，并覆盖其中特定方法并添加增强代码，从而实现AOP。CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的。



## 通知

### 通知类型

- **前置通知(Before Advice)**：在目标方法被调用前调用通知功能；
- **后置通知(After Advice)**：在目标方法被调用之后调用通知功能；
- **返回通知(After-returning)**：在目标方法成功执行之后调用通知功能；
- **异常通知(After-throwing)**：在目标方法抛出异常之后调用通知功能；
- **环绕通知(Around)**：把整个目标方法包裹起来，在被调用前和调用之后分别调用通知功能。



#### 前置通知

#### 后置通知

#### 返回通知

#### 异常通知

#### 环绕通知



### 通知执行顺序

#### 正常情况

around before > before > target method 执行 > around after > after > afterReturning



#### 异常情况

around before > before > target method 执行 > around after > after >  afterThrowing



## 工作流程

```java
@Data
@Service
public class Louzai {
    public void everyDay() {
        System.out.println("睡觉");
    }
}
```



```java
@Aspect
@Component
public class LouzaiAspect {
    
    @Pointcut("execution(* com.java.Louzai.everyDay())")
    private void myPointCut() {
    }

    // 前置通知
    @Before("myPointCut()")
    public void myBefore() {
        System.out.println("吃饭");
    }

    // 后置通知
    @AfterReturning(value = "myPointCut()")
    public void myAfterReturning() {
        System.out.println("打豆豆。。。");
    }
}
```



```java
public class MyTest {
    public static void main(String[] args) {
        ApplicationContext context =new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
        Louzai louzai = (Louzai) context.getBean("louzai");
        louzai.everyDay();
    }
}
```



<img src="https://oss.xubighead.top/oss/image/202506/1930089286805327873.jpg" alt="图片" style="zoom: 50%;" />



第一块就是**前置处理**，我们在创建 Louzai Bean 的前置处理中，会遍历程序所有的切面信息，然后**将切面信息保存在缓存中**，比如示例中 LouzaiAspect 的所有切面信息。

第二块就是**后置处理**，我们在创建 Louzai Bean 的后置处理器中，里面会做两件事情：

- **获取 Louzai 的切面方法**：首先会从缓存中拿到所有的切面信息，和 Louzai 的所有方法进行匹配，然后找到 Louzai 所有需要进行 AOP 的方法。
- **创建 AOP 代理对象**：结合 Louzai 需要进行 AOP 的方法，选择 Cglib 或 JDK，创建 AOP 代理对象。



### 代码入口

> org.springframework.context.support.AbstractApplicationContext

```java
@Override
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
       	// ...

        try {
            // ...

            // 实例化所有Bean工程缓存的Bean对象
            finishBeanFactoryInitialization(beanFactory);

            // ...
        }
		// ...
    }
}

protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    // ...

    // 实例化剩下的单例对象
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
    else {
        // ...
        try {
            // ...
            // Create bean instance.
            if (mbd.isSingleton()) {
                sharedInstance = getSingleton(beanName, () -> {
                    try {
                        // 创建单例bean
                        return createBean(beanName, mbd, args);
                    }
                    // ...
                });
                bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
            }
			// ...
        }
        // ...
    }
    // ...
    return (T) bean;
}
```



### 前置处理

<img src="https://oss.xubighead.top/oss/image/202506/1930090675027677185.jpg" alt="图片" style="zoom:50%;" />



> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
@Override
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
    throws BeanCreationException {
	// ...

    try {
        // 前置处理入口
        Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
        if (bean != null) {
            return bean;
        }
    }
    // ...
}

@Nullable
protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
    Object bean = null;
    if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
        if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
            Class<?> targetType = determineTargetType(beanName, mbd);
            if (targetType != null) {
                // 前置处理
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

@Nullable
protected Object applyBeanPostProcessorsBeforeInstantiation(Class<?> beanClass, String beanName) {
    for (BeanPostProcessor bp : getBeanPostProcessors()) {
        if (bp instanceof InstantiationAwareBeanPostProcessor) {
            InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
            // 遍历到AnnotationAwareAspectJAutoProxyCreator进入下一步
            Object result = ibp.postProcessBeforeInstantiation(beanClass, beanName);
            if (result != null) {
                return result;
            }
        }
    }
    return null;
}
```



AnnotationAwareAspectJAutoProxyCreator继承了AbstractAutoProxyCreator，调用该抽象类实现的postProcessBeforeInstantiation方法。



> org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator

```java
@Override
public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) {
    Object cacheKey = getCacheKey(beanClass, beanName);

    if (!StringUtils.hasLength(beanName) || !this.targetSourcedBeans.contains(beanName)) {
        if (this.advisedBeans.containsKey(cacheKey)) {
            return null;
        }
        if (isInfrastructureClass(beanClass) || shouldSkip(beanClass, beanName)) {
            this.advisedBeans.put(cacheKey, Boolean.FALSE);
            return null;
        }
    }
    // ...
    return null;
}
```



> org.springframework.aop.aspectj.autoproxy.AspectJAwareAdvisorAutoProxyCreator

```java
@Override
protected boolean shouldSkip(Class<?> beanClass, String beanName) {
    // 获取切面信息
    List<Advisor> candidateAdvisors = findCandidateAdvisors();
    for (Advisor advisor : candidateAdvisors) {
        if (advisor instanceof AspectJPointcutAdvisor &&
            ((AspectJPointcutAdvisor) advisor).getAspectName().equals(beanName)) {
            return true;
        }
    }
    return super.shouldSkip(beanClass, beanName);
}
```



> org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator

```java
@Override
protected List<Advisor> findCandidateAdvisors() {
    // Add all the Spring advisors found according to superclass rules.
    List<Advisor> advisors = super.findCandidateAdvisors();
    // Build Advisors for all AspectJ aspects in the bean factory.
    if (this.aspectJAdvisorsBuilder != null) {
        advisors.addAll(this.aspectJAdvisorsBuilder.buildAspectJAdvisors());
    }
    return advisors;
}
```



1. 先遍历所有的类；
2. 判断是否切面，只有切面才会进入后面逻辑；
3. 获取每个 Aspect 的切面列表；
4. 保存 Aspect 的切面列表到缓存 advisorsCache 中。



> org.springframework.aop.aspectj.annotation.BeanFactoryAspectJAdvisorsBuilder

```java
public List<Advisor> buildAspectJAdvisors() {
    List<String> aspectNames = this.aspectBeanNames;

    if (aspectNames == null) {
        synchronized (this) {
            aspectNames = this.aspectBeanNames;
            if (aspectNames == null) {
                List<Advisor> advisors = new ArrayList<>();
                aspectNames = new ArrayList<>();
                // 遍历
                String[] beanNames = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(
                    this.beanFactory, Object.class, true, false);
                for (String beanName : beanNames) {
                    if (!isEligibleBean(beanName)) {
                        continue;
                    }
                    // We must be careful not to instantiate beans eagerly as in this case they
                    // would be cached by the Spring container but would not have been weaved.
                    Class<?> beanType = this.beanFactory.getType(beanName);
                    if (beanType == null) {
                        continue;
                    }
                    // 判断是否是切面
                    if (this.advisorFactory.isAspect(beanType)) {
                        aspectNames.add(beanName);
                        AspectMetadata amd = new AspectMetadata(beanType, beanName);
                        if (amd.getAjType().getPerClause().getKind() == PerClauseKind.SINGLETON) {
                            MetadataAwareAspectInstanceFactory factory =
                                new BeanFactoryAspectInstanceFactory(this.beanFactory, beanName);
                            // 获取所有的切面信息
                            List<Advisor> classAdvisors = this.advisorFactory.getAdvisors(factory);
                            if (this.beanFactory.isSingleton(beanName)) {
                                // 保存切面信息
                                this.advisorsCache.put(beanName, classAdvisors);
                            }
                            else {
                                this.aspectFactoryCache.put(beanName, factory);
                            }
                            advisors.addAll(classAdvisors);
                        }
                        else {
                            // Per target or per this.
                            if (this.beanFactory.isSingleton(beanName)) {
                                throw new IllegalArgumentException("Bean with name '" + beanName +
                                                                   "' is a singleton, but aspect instantiation model is not singleton");
                            }
                            MetadataAwareAspectInstanceFactory factory =
                                new PrototypeAspectInstanceFactory(this.beanFactory, beanName);
                            this.aspectFactoryCache.put(beanName, factory);
                            advisors.addAll(this.advisorFactory.getAdvisors(factory));
                        }
                    }
                }
                this.aspectBeanNames = aspectNames;
                return advisors;
            }
        }
    }

    if (aspectNames.isEmpty()) {
        return Collections.emptyList();
    }
    List<Advisor> advisors = new ArrayList<>();
    for (String aspectName : aspectNames) {
        List<Advisor> cachedAdvisors = this.advisorsCache.get(aspectName);
        if (cachedAdvisors != null) {
            advisors.addAll(cachedAdvisors);
        }
        else {
            MetadataAwareAspectInstanceFactory factory = this.aspectFactoryCache.get(aspectName);
            advisors.addAll(this.advisorFactory.getAdvisors(factory));
        }
    }
    return advisors;
}
```



#### 判断是否是切面

> org.springframework.aop.aspectj.annotation.AbstractAspectJAdvisorFactory

```java
@Override
public boolean isAspect(Class<?> clazz) {
    return (hasAspectAnnotation(clazz) && !compiledByAjc(clazz));
}

private boolean hasAspectAnnotation(Class<?> clazz) {
    return (AnnotationUtils.findAnnotation(clazz, Aspect.class) != null);
}
```



#### 获取切面列表

> org.springframework.aop.aspectj.annotation.ReflectiveAspectJAdvisorFactory

```java
@Override
public List<Advisor> getAdvisors(MetadataAwareAspectInstanceFactory aspectInstanceFactory) {
    Class<?> aspectClass = aspectInstanceFactory.getAspectMetadata().getAspectClass();
    String aspectName = aspectInstanceFactory.getAspectMetadata().getAspectName();
    validate(aspectClass);

    // We need to wrap the MetadataAwareAspectInstanceFactory with a decorator
    // so that it will only instantiate once.
    MetadataAwareAspectInstanceFactory lazySingletonAspectInstanceFactory =
        new LazySingletonAspectInstanceFactoryDecorator(aspectInstanceFactory);

    List<Advisor> advisors = new ArrayList<>();
    // 遍历方法列表
    for (Method method : getAdvisorMethods(aspectClass)) {
        Advisor advisor = getAdvisor(method, lazySingletonAspectInstanceFactory, advisors.size(), aspectName);
        if (advisor != null) {
            advisors.add(advisor);
        }
    }

    // If it's a per target aspect, emit the dummy instantiating aspect.
    if (!advisors.isEmpty() && lazySingletonAspectInstanceFactory.getAspectMetadata().isLazilyInstantiated()) {
        Advisor instantiationAdvisor = new SyntheticInstantiationAdvisor(lazySingletonAspectInstanceFactory);
        advisors.add(0, instantiationAdvisor);
    }

    // Find introduction fields.
    for (Field field : aspectClass.getDeclaredFields()) {
        Advisor advisor = getDeclareParentsAdvisor(field);
        if (advisor != null) {
            advisors.add(advisor);
        }
    }

    return advisors;
}

@Override
@Nullable
public Advisor getAdvisor(Method candidateAdviceMethod, MetadataAwareAspectInstanceFactory aspectInstanceFactory,
                          int declarationOrderInAspect, String aspectName) {

    validate(aspectInstanceFactory.getAspectMetadata().getAspectClass());

    AspectJExpressionPointcut expressionPointcut = getPointcut(
        candidateAdviceMethod, aspectInstanceFactory.getAspectMetadata().getAspectClass());
    if (expressionPointcut == null) {
        return null;
    }

    return new InstantiationModelAwarePointcutAdvisorImpl(expressionPointcut, candidateAdviceMethod,
                                                          this, aspectInstanceFactory, declarationOrderInAspect, aspectName);
}
```



> org.springframework.aop.aspectj.annotation.InstantiationModelAwarePointcutAdvisorImpl

```java
public InstantiationModelAwarePointcutAdvisorImpl(AspectJExpressionPointcut declaredPointcut,
                                                  Method aspectJAdviceMethod, AspectJAdvisorFactory aspectJAdvisorFactory,
                                                  MetadataAwareAspectInstanceFactory aspectInstanceFactory, int declarationOrder, String aspectName) {

    this.declaredPointcut = declaredPointcut;
    this.declaringClass = aspectJAdviceMethod.getDeclaringClass();
    this.methodName = aspectJAdviceMethod.getName();
    this.parameterTypes = aspectJAdviceMethod.getParameterTypes();
    this.aspectJAdviceMethod = aspectJAdviceMethod;
    this.aspectJAdvisorFactory = aspectJAdvisorFactory;
    this.aspectInstanceFactory = aspectInstanceFactory;
    this.declarationOrder = declarationOrder;
    this.aspectName = aspectName;

    if (aspectInstanceFactory.getAspectMetadata().isLazilyInstantiated()) {
        // Static part of the pointcut is a lazy type.
        Pointcut preInstantiationPointcut = Pointcuts.union(
            aspectInstanceFactory.getAspectMetadata().getPerClausePointcut(), this.declaredPointcut);

        // Make it dynamic: must mutate from pre-instantiation to post-instantiation state.
        // If it's not a dynamic pointcut, it may be optimized out
        // by the Spring AOP infrastructure after the first evaluation.
        this.pointcut = new PerTargetInstantiationModelPointcut(
            this.declaredPointcut, preInstantiationPointcut, aspectInstanceFactory);
        this.lazy = true;
    }
    else {
        // A singleton aspect.
        this.pointcut = this.declaredPointcut;
        this.lazy = false;
        this.instantiatedAdvice = instantiateAdvice(this.declaredPointcut);
    }
}

private Advice instantiateAdvice(AspectJExpressionPointcut pointcut) {
    Advice advice = this.aspectJAdvisorFactory.getAdvice(this.aspectJAdviceMethod, pointcut,
                                                         this.aspectInstanceFactory, this.declarationOrder, this.aspectName);
    return (advice != null ? advice : EMPTY_ADVICE);
}
```



> org.springframework.aop.aspectj.annotation.ReflectiveAspectJAdvisorFactory

```java
@Override
@Nullable
public Advice getAdvice(Method candidateAdviceMethod, AspectJExpressionPointcut expressionPointcut,
                        MetadataAwareAspectInstanceFactory aspectInstanceFactory, int declarationOrder, String aspectName) {

    Class<?> candidateAspectClass = aspectInstanceFactory.getAspectMetadata().getAspectClass();
    validate(candidateAspectClass);

    AspectJAnnotation<?> aspectJAnnotation =
        AbstractAspectJAdvisorFactory.findAspectJAnnotationOnMethod(candidateAdviceMethod);
    if (aspectJAnnotation == null) {
        return null;
    }

    // If we get here, we know we have an AspectJ method.
    // Check that it's an AspectJ-annotated class
    if (!isAspect(candidateAspectClass)) {
        throw new AopConfigException("Advice must be declared inside an aspect type: " +
                                     "Offending method '" + candidateAdviceMethod + "' in class [" +
                                     candidateAspectClass.getName() + "]");
    }

    if (logger.isDebugEnabled()) {
        logger.debug("Found AspectJ method: " + candidateAdviceMethod);
    }

    AbstractAspectJAdvice springAdvice;

    switch (aspectJAnnotation.getAnnotationType()) {
        case AtPointcut:
            if (logger.isDebugEnabled()) {
                logger.debug("Processing pointcut '" + candidateAdviceMethod.getName() + "'");
            }
            return null;
        case AtAround:
            springAdvice = new AspectJAroundAdvice(
                candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
            break;
        case AtBefore:
            // 生成切面信息
            springAdvice = new AspectJMethodBeforeAdvice(
                candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
            break;
        case AtAfter:
            springAdvice = new AspectJAfterAdvice(
                candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
            break;
        case AtAfterReturning:
            springAdvice = new AspectJAfterReturningAdvice(
                candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
            AfterReturning afterReturningAnnotation = (AfterReturning) aspectJAnnotation.getAnnotation();
            if (StringUtils.hasText(afterReturningAnnotation.returning())) {
                springAdvice.setReturningName(afterReturningAnnotation.returning());
            }
            break;
        case AtAfterThrowing:
            springAdvice = new AspectJAfterThrowingAdvice(
                candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
            AfterThrowing afterThrowingAnnotation = (AfterThrowing) aspectJAnnotation.getAnnotation();
            if (StringUtils.hasText(afterThrowingAnnotation.throwing())) {
                springAdvice.setThrowingName(afterThrowingAnnotation.throwing());
            }
            break;
        default:
            throw new UnsupportedOperationException(
                "Unsupported advice type on method: " + candidateAdviceMethod);
    }

    // Now to configure the advice...
    springAdvice.setAspectName(aspectName);
    springAdvice.setDeclarationOrder(declarationOrder);
    String[] argNames = this.parameterNameDiscoverer.getParameterNames(candidateAdviceMethod);
    if (argNames != null) {
        springAdvice.setArgumentNamesFromStringArray(argNames);
    }
    springAdvice.calculateArgumentBindings();

    return springAdvice;
}

```



### 后置处理

![图片](https://oss.xubighead.top/oss/image/202506/1930095815197560833.jpg)



> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
@Override
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
    throws BeanCreationException {
    // ...
    try {
        Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
        if (bean != null) {
            return bean;
        }
    }
    // ...

    try {
        // 入口
        Object beanInstance = doCreateBean(beanName, mbdToUse, args);
        // ...
        return beanInstance;
    }
    // ...
}

protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
    throws BeanCreationException {
    // ...
    Object exposedObject = bean;
    try {
        populateBean(beanName, mbd, instanceWrapper);
        exposedObject = initializeBean(beanName, exposedObject, mbd);
    }
    // ...
    return exposedObject;
}

protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
	// ...
    if (mbd == null || !mbd.isSynthetic()) {
        // 后置处理
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    }

    return wrappedBean;
}

@Override
public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
    throws BeansException {

    Object result = existingBean;
    for (BeanPostProcessor processor : getBeanPostProcessors()) {
        // 遍历至对象AnnotationAwareAspectJAutoProxyCreator
        Object current = processor.postProcessAfterInitialization(result, beanName);
        if (current == null) {
            return result;
        }
        result = current;
    }
    return result;
}
```



AnnotationAwareAspectJAutoProxyCreator继承了AbstractAutoProxyCreator抽象类。



> org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator

```java
@Override
public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) {
    if (bean != null) {
        Object cacheKey = getCacheKey(bean.getClass(), beanName);
        if (this.earlyProxyReferences.remove(cacheKey) != bean) {
            return wrapIfNecessary(bean, beanName, cacheKey);
        }
    }
    return bean;
}

protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
    if (StringUtils.hasLength(beanName) && this.targetSourcedBeans.contains(beanName)) {
        return bean;
    }
    if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
        return bean;
    }
    if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
        this.advisedBeans.put(cacheKey, Boolean.FALSE);
        return bean;
    }

    // 获取所有切面列表
    Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
    if (specificInterceptors != DO_NOT_PROXY) {
        this.advisedBeans.put(cacheKey, Boolean.TRUE);
        // 创建代理对象
        Object proxy = createProxy(
            bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
        this.proxyTypes.put(cacheKey, proxy.getClass());
        return proxy;
    }

    this.advisedBeans.put(cacheKey, Boolean.FALSE);
    return bean;
}
```



#### 获取切面

> org.springframework.aop.framework.autoproxy.AbstractAdvisorAutoProxyCreator

```java
@Override
@Nullable
protected Object[] getAdvicesAndAdvisorsForBean(
    Class<?> beanClass, String beanName, @Nullable TargetSource targetSource) {

    List<Advisor> advisors = findEligibleAdvisors(beanClass, beanName);
    if (advisors.isEmpty()) {
        return DO_NOT_PROXY;
    }
    return advisors.toArray();
}

protected List<Advisor> findEligibleAdvisors(Class<?> beanClass, String beanName) {
    List<Advisor> candidateAdvisors = findCandidateAdvisors();
    List<Advisor> eligibleAdvisors = findAdvisorsThatCanApply(candidateAdvisors, beanClass, beanName);
    extendAdvisors(eligibleAdvisors);
    if (!eligibleAdvisors.isEmpty()) {
        eligibleAdvisors = sortAdvisors(eligibleAdvisors);
    }
    return eligibleAdvisors;
}
```



> org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator

```java
@Override
protected List<Advisor> findCandidateAdvisors() {
    // Add all the Spring advisors found according to superclass rules.
    List<Advisor> advisors = super.findCandidateAdvisors();
    // Build Advisors for all AspectJ aspects in the bean factory.
    if (this.aspectJAdvisorsBuilder != null) {
        advisors.addAll(this.aspectJAdvisorsBuilder.buildAspectJAdvisors());
    }
    return advisors;
}
```



进入 buildAspectJAdvisors()，这个方法应该有印象，就是前面将切面信息放入缓存 advisorsCache 中，现在这里就是要获取缓存。



> org.springframework.aop.aspectj.annotation.BeanFactoryAspectJAdvisorsBuilder

```java
public List<Advisor> buildAspectJAdvisors() {
    List<String> aspectNames = this.aspectBeanNames;
	// ...
    List<Advisor> advisors = new ArrayList<>();
    for (String aspectName : aspectNames) {
        // 缓存中获取切面列表
        List<Advisor> cachedAdvisors = this.advisorsCache.get(aspectName);
        if (cachedAdvisors != null) {
            advisors.addAll(cachedAdvisors);
        }
        else {
            MetadataAwareAspectInstanceFactory factory = this.aspectFactoryCache.get(aspectName);
            advisors.addAll(this.advisorFactory.getAdvisors(factory));
        }
    }
    return advisors;
}
```



再回到 findEligibleAdvisors()，从缓存拿到所有的切面信息后，继续往后执行。

> org.springframework.aop.framework.autoproxy.AbstractAdvisorAutoProxyCreator

```java
protected List<Advisor> findAdvisorsThatCanApply(
    List<Advisor> candidateAdvisors, Class<?> beanClass, String beanName) {

    ProxyCreationContext.setCurrentProxiedBeanName(beanName);
    try {
        return AopUtils.findAdvisorsThatCanApply(candidateAdvisors, beanClass);
    }
    finally {
        ProxyCreationContext.setCurrentProxiedBeanName(null);
    }
}
```



> org.springframework.aop.support.AopUtils

```java
public static List<Advisor> findAdvisorsThatCanApply(List<Advisor> candidateAdvisors, Class<?> clazz) {
    if (candidateAdvisors.isEmpty()) {
        return candidateAdvisors;
    }
    List<Advisor> eligibleAdvisors = new ArrayList<>();
    for (Advisor candidate : candidateAdvisors) {
        if (candidate instanceof IntroductionAdvisor && canApply(candidate, clazz)) {
            eligibleAdvisors.add(candidate);
        }
    }
    boolean hasIntroductions = !eligibleAdvisors.isEmpty();
    // 遍历每一个增强器，看是否可以作用于被增强类
    for (Advisor candidate : candidateAdvisors) {
        if (candidate instanceof IntroductionAdvisor) {
            // already processed
            continue;
        }
        if (canApply(candidate, clazz, hasIntroductions)) {
            eligibleAdvisors.add(candidate);
        }
    }
    return eligibleAdvisors;
}

public static boolean canApply(Advisor advisor, Class<?> targetClass, boolean hasIntroductions) {
    if (advisor instanceof IntroductionAdvisor) {
        return ((IntroductionAdvisor) advisor).getClassFilter().matches(targetClass);
    }
    else if (advisor instanceof PointcutAdvisor) {
        PointcutAdvisor pca = (PointcutAdvisor) advisor;
        return canApply(pca.getPointcut(), targetClass, hasIntroductions);
    }
    else {
        // It doesn't have a pointcut so we assume it applies.
        return true;
    }
}	

public static boolean canApply(Pointcut pc, Class<?> targetClass, boolean hasIntroductions) {
    Assert.notNull(pc, "Pointcut must not be null");
    if (!pc.getClassFilter().matches(targetClass)) {
        return false;
    }

    MethodMatcher methodMatcher = pc.getMethodMatcher();
    if (methodMatcher == MethodMatcher.TRUE) {
        // No need to iterate the methods if we're matching any method anyway...
        return true;
    }

    IntroductionAwareMethodMatcher introductionAwareMethodMatcher = null;
    if (methodMatcher instanceof IntroductionAwareMethodMatcher) {
        introductionAwareMethodMatcher = (IntroductionAwareMethodMatcher) methodMatcher;
    }

    Set<Class<?>> classes = new LinkedHashSet<>();
    if (!Proxy.isProxyClass(targetClass)) {
        classes.add(ClassUtils.getUserClass(targetClass));
    }
    classes.addAll(ClassUtils.getAllInterfacesForClassAsSet(targetClass));

    for (Class<?> clazz : classes) {
        // 遍历被AOP所有方法，用增强器进行匹配
        Method[] methods = ReflectionUtils.getAllDeclaredMethods(clazz);
        for (Method method : methods) {
            if (introductionAwareMethodMatcher != null ?
                introductionAwareMethodMatcher.matches(method, targetClass, hasIntroductions) :
                methodMatcher.matches(method, targetClass)) {
                return true;
            }
        }
    }

    return false;
}
```



#### 创建代理对象

> org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator

```java
protected Object createProxy(Class<?> beanClass, @Nullable String beanName,
                             @Nullable Object[] specificInterceptors, TargetSource targetSource) {

    if (this.beanFactory instanceof ConfigurableListableBeanFactory) {
        AutoProxyUtils.exposeTargetClass((ConfigurableListableBeanFactory) this.beanFactory, beanName, beanClass);
    }

    ProxyFactory proxyFactory = new ProxyFactory();
    proxyFactory.copyFrom(this);

    if (!proxyFactory.isProxyTargetClass()) {
        if (shouldProxyTargetClass(beanClass, beanName)) {
            proxyFactory.setProxyTargetClass(true);
        }
        else {
            evaluateProxyInterfaces(beanClass, proxyFactory);
        }
    }

    Advisor[] advisors = buildAdvisors(beanName, specificInterceptors);
    // 设置切面列表
    proxyFactory.addAdvisors(advisors);
    proxyFactory.setTargetSource(targetSource);
    customizeProxyFactory(proxyFactory);

    proxyFactory.setFrozen(this.freezeProxy);
    if (advisorsPreFiltered()) {
        proxyFactory.setPreFiltered(true);
    }
    // 创建代理对象
    return proxyFactory.getProxy(getProxyClassLoader());
}
```



> org.springframework.aop.framework.ProxyFactory

```java
public Object getProxy(@Nullable ClassLoader classLoader) {
    return createAopProxy().getProxy(classLoader);
}
```



> org.springframework.aop.framework.ProxyCreatorSupport

```java
protected final synchronized AopProxy createAopProxy() {
    if (!this.active) {
        activate();
    }
    return getAopProxyFactory().createAopProxy(this);
}
```



> org.springframework.aop.framework.DefaultAopProxyFactory

```java
@Override
public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
    if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
        Class<?> targetClass = config.getTargetClass();
        if (targetClass == null) {
            throw new AopConfigException("TargetSource cannot determine target class: " +
                                         "Either an interface or a target is required for proxy creation.");
        }
        if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
            return new JdkDynamicAopProxy(config);
        }
        return new ObjenesisCglibAopProxy(config);
    }
    else {
        return new JdkDynamicAopProxy(config);
    }
}
```



### 切面执行

![图片](https://oss.xubighead.top/oss/image/202506/1930095969015271426.jpg)



切面执行入口如下：

```java
@Override
@Nullable
public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
    // ...
    try {
        // ...
        else {
            // We need to create a method invocation...
            retVal = new CglibMethodInvocation(proxy, target, method, args, targetClass, chain, methodProxy).proceed();
        }
        // ...
    }
    // ...
}
```



下面就是“执行切面”最核心的逻辑设计思路：

1. **设计思路**：采用递归 + 责任链的模式；
2. **递归**：反复执行 CglibMethodInvocation 的 proceed()；
3. **退出递归条件**：interceptorsAndDynamicMethodMatchers 数组中的对象，全部执行完毕；
4. **责任链**：示例中的责任链，是个长度为 3 的数组，每次取其中一个数组对象，然后去执行对象的 invoke（）。



> org.springframework.aop.framework.ReflectiveMethodInvocation

```java
@Override
@Nullable
public Object proceed() throws Throwable {
    // 递归执行
    if (this.currentInterceptorIndex == this.interceptorsAndDynamicMethodMatchers.size() - 1) {
        // 退出递归
        return invokeJoinpoint();
    }

    Object interceptorOrInterceptionAdvice =
        this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
    if (interceptorOrInterceptionAdvice instanceof InterceptorAndDynamicMethodMatcher) {
        InterceptorAndDynamicMethodMatcher dm =
            (InterceptorAndDynamicMethodMatcher) interceptorOrInterceptionAdvice;
        Class<?> targetClass = (this.targetClass != null ? this.targetClass : this.method.getDeclaringClass());
        if (dm.methodMatcher.matches(this.method, targetClass, this.arguments)) {
            return dm.interceptor.invoke(this);
        }
        else {
            return proceed();
        }
    }
    else {
        // 责任链执行
        return ((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this);
    }
}
```



因为数组里面只有 3 个对象，所以只会递归 3 次。



#### 第一次递归

数组的第一个对象是 ExposeInvocationInterceptor，执行 invoke（），**注意入参是 CglibMethodInvocation。**



> org.springframework.aop.interceptor.ExposeInvocationInterceptor

```java
@Override
public Object invoke(MethodInvocation mi) throws Throwable {
    MethodInvocation oldInvocation = invocation.get();
    invocation.set(mi);
    try {
        return mi.proceed();
    }
    finally {
        invocation.set(oldInvocation);
    }
}
```



里面啥都没干，继续执行CglibMethodInvocation的process方法，此时interceptorOrInterceptionAdvice指向下标为1的对象。



#### 第二次递归

数组的第二个对象是 MethodBeforeAdviceInterceptor，执行 invoke。



> org.springframework.aop.framework.adapter.MethodBeforeAdviceInterceptor

```java
@Override
public Object invoke(MethodInvocation mi) throws Throwable {
    // 执行切面
    this.advice.before(mi.getMethod(), mi.getArguments(), mi.getThis());
    // 继续递归
    return mi.proceed();
}
```



> org.springframework.aop.aspectj.AspectJMethodBeforeAdvice

```java
@Override
public void before(Method method, Object[] args, @Nullable Object target) throws Throwable {
    invokeAdviceMethod(getJoinPointMatch(), null, null);
}
```



> org.springframework.aop.aspectj.AbstractAspectJAdvice

```java
protected Object invokeAdviceMethod(
    @Nullable JoinPointMatch jpMatch, @Nullable Object returnValue, @Nullable Throwable ex)
    throws Throwable {

    return invokeAdviceMethodWithGivenArgs(argBinding(getJoinPoint(), jpMatch, returnValue, ex));
}

protected Object invokeAdviceMethodWithGivenArgs(Object[] args) throws Throwable {
    Object[] actualArgs = args;
    if (this.aspectJAdviceMethod.getParameterCount() == 0) {
        actualArgs = null;
    }
    try {
        ReflectionUtils.makeAccessible(this.aspectJAdviceMethod);
        // 执行前置切面
        return this.aspectJAdviceMethod.invoke(this.aspectInstanceFactory.getAspectInstance(), actualArgs);
    }
    catch (IllegalArgumentException ex) {
        throw new AopInvocationException("Mismatch on arguments to advice method [" +
                                         this.aspectJAdviceMethod + "]; pointcut expression [" +
                                         this.pointcut.getPointcutExpression() + "]", ex);
    }
    catch (InvocationTargetException ex) {
        throw ex.getTargetException();
    }
}
```



#### 第三次递归

数组的第二个对象是 AfterReturningAdviceInterceptor，执行 invoke。



> org.springframework.aop.framework.adapter.AfterReturningAdviceInterceptor

```java
@Override
public Object invoke(MethodInvocation mi) throws Throwable {
    Object retVal = mi.proceed();
    this.advice.afterReturning(retVal, mi.getMethod(), mi.getArguments(), mi.getThis());
    return retVal;
}
```



> org.springframework.aop.aspectj.AspectJAfterReturningAdvice

```java
@Override
public void afterReturning(@Nullable Object returnValue, Method method, Object[] args, @Nullable Object target) throws Throwable {
    if (shouldInvokeOnReturnValueOf(method, returnValue)) {
        invokeAdviceMethod(getJoinPointMatch(), returnValue, null);
    }
}
```



#### 退出递归

第四次进入proceed方法时就会退出递归。



> org.springframework.aop.framework.ReflectiveMethodInvocation

```java
@Override
@Nullable
public Object proceed() throws Throwable {
    // 递归执行
    if (this.currentInterceptorIndex == this.interceptorsAndDynamicMethodMatchers.size() - 1) {
        // 退出递归
        return invokeJoinpoint();
    }
	// ...
}
```



执行完上面逻辑，就会退出递归，invokeJoinpoint方法的执行逻辑，其实就是执行主方法。

> org.springframework.aop.framework.CglibAopProxy

```java
@Override
protected Object invokeJoinpoint() throws Throwable {
    if (this.methodProxy != null) {
        return this.methodProxy.invoke(this.target, this.arguments);
    }
    else {
        return super.invokeJoinpoint();
    }
}
```



#### 切面执行总结

主对象是 CglibMethodInvocation，继承于 ReflectiveMethodInvocation，然后 process() 的核心逻辑，其实都在 ReflectiveMethodInvocation 中。

ReflectiveMethodInvocation 中的 process() 方法，里面有个长度为 3 的数组 interceptorsAndDynamicMethodMatchers，里面存储了 3 个对象，分别为 

- ExposeInvocationInterceptor
- MethodBeforeAdviceInterceptor
- AfterReturningAdviceInterceptor。



对象和方法的关系：

- **接口继承**：数组中的 3 个对象，都是继承 MethodInterceptor 接口，实现里面的 invoke() 方法；
- **类继承**：我们的主对象 CglibMethodInvocation，继承于 ReflectiveMethodInvocation，复用它的 process() 方法；
- **两者结合（策略模式）**：invoke() 的入参，就是 CglibMethodInvocation，执行 invoke() 时，内部会执行 CglibMethodInvocation.process()，这个其实就是个策略模式。



执行逻辑：

- **程序入口**：是 CglibMethodInvocation 的 process() 方法；
- **链式执行（衍生的责任链模式）**：process() 中有个包含 3 个对象的数组，依次去执行每个对象的 invoke() 方法。
- **递归（逻辑回退）**：invoke() 方法会执行切面逻辑，同时也会执行 CglibMethodInvocation 的 process() 方法，让逻辑再一次进入 process()。
- **递归退出**：当数字中的 3 个对象全部执行完毕，流程结束。



数组里面的 3 个对象，里面没有 next 成员对象，所以不能直接用责任链模式，单独搞了一个 CglibMethodInvocation.process()，通过去无限递归 process()，来实现这个责任链的逻辑。



### 总结

AOP 的源码执行逻辑，分为 4 块：

- 将所有的切面都保存在缓存中；
- 取出缓存中的切面列表，和对象的所有方法匹配，拿到属于对象的切面列表；
- 创建 AOP 代理对象；
- 通过“责任链 + 递归”，去执行切面逻辑。