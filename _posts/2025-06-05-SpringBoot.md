---
layout: post
title: 第018章-SpringBoot
categories: [Spring]
description: 
keywords: SpringBoot.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# Spring_Boot

Spring Boot是Spring开源组织下的子项目，是Spring组件一站式解决方案，主要是简化了使用Spring的难度，简省了繁重的配置，提供了各种启动器，开发者能快速上手。



## Introduction

### SpringBoot Feature

1. 容易上手，提升开发效率，为 Spring 开发提供一个更快、更广泛的入门体验。
2. 开箱即用，远离繁琐的配置。
3. 提供了一系列大型项目通用的非业务性功能，例如：内嵌服务器、安全管理、运行数据监控、运行状况检查和外部化配置等。
4. 没有代码生成，也不需要XML配置。
5. 避免大量的 Maven 导入和各种版本冲突。



- 为了解决java开发中的，繁多的配置、底下的开发效率，复杂的部署流程，和第三方技术集成难度大的问题，产生了spring boot。
- springboot 使用 “习惯优于配置”的理念让项目快速运行起来，使用springboot很容易创建一个独立运行的jar，内嵌servlet容器
- springboot的核心功能一：独立运行spring项目，springboot可以以jar包的形式独立运行，运行一个springboot项目只需要 java -jar xxx.jar 来运行
- springboot的核心功能二：内嵌servlet容器，可以内嵌tomcat，接天jetty，或者undertow，这样我们就可以不用war包形式部署项目
- springboot的核心功能三，提供starter简化maven配置，spring提供了一系列starter pom 来简化maven的依赖加载， 当使用了 spring-boot-starter-web时，会自动加载所需要的依赖包
- springboot的核心功能三：自动配置spring sprintboot 会根据在类路径的jar包，类，为jar包中的类自动配置bean，这样会极大的减少使用的配置，会根据启动类所在的目录，自动配置bean



### 运行方式

1）打包用命令或者放到容器中运行

2）用 Maven/ Gradle 插件运行

3）直接执行 main 方法运行



## Annotation

### Related Annotation

启动类上面的注解是@SpringBootApplication，它也是 Spring Boot 的核心注解，主要组合包含了以下 3 个注解：

@SpringBootConfiguration：组合了 @Configuration 注解，实现配置文件的功能。

@EnableAutoConfiguration：打开自动配置的功能，也可以关闭某个自动配置的选项，如关闭数据源自动配置功能： @SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })。

@ComponentScan：Spring组件扫描。



#### @SpringBootApplication

`@SpringBootApplication`注解是一个快捷的配置注解，在被它标注的类中，可以定义一个或多个Bean，并自动触发自动配置Bean和自动扫描组件。此注解相当于`@Configuration`、`@EnableAutoConfiguration`和`@ComponentScan`的组合。

扫描包过程中，Spring允许用户按照条件过滤所需要的Bean，@SpringBootApplication中本身包含了`@ComponentScan`注解，并为注解配置了两个Filter：TypeExcludeFilter和AutoConfigurationExcludeFilter注解。这两个注解允许用户实现特定的类，从而实现对Bean定义的过滤。



#### @SpringBootConfiguration

@SpringBootConfiguration该注解的作用是用来指定扫描路径的，如果不指定特定的扫描路径的话，扫描的路径是当前修饰的类所在的包及其子包。

@SpringBootConfiguration这个注解的本质其实是@Configuration注解。	



```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {

}
```



#### @EnableAutoConfiguration

@EnableAutoConfiguration注解用于通知Spring，根据当前类路径下引入的依赖包，自动配置与这些依赖包相关的配置项。



#### @AutoConfigurationPackage

@AutoConfigurationPackage用于将添加该注解的类所在的package作为自动配置package进行管理，这个注解依旧是通过@Import注解向容器中注册Bean。

> org.springframework.boot.autoconfigure.AutoConfigurationPackage

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage {

}
```



注解导入了Registrar.class，其本质是一个ImportBeanDefinitionRegistrar，会把当前注解类所在的包注入到Spring容器中。

> org.springframework.boot.autoconfigure.AutoConfigurationPackages.Registrar

```java
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {

    @Override
    public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
        register(registry, new PackageImport(metadata).getPackageName());
    }

    @Override
    public Set<Object> determineImports(AnnotationMetadata metadata) {
        return Collections.singleton(new PackageImport(metadata));
    }

}
```

@ComponentScan并不会把类所在的包注入到容器中，@ComponentScan只注入指定的包。类所在的包通过@AutoConfigurationPackage注入。



### Commonly Use Annotation

#### @ConditionalOnBean

#### @ConditionalOnMissingBean

这两个注解属于对象条件注解，根据是否存在某个对象作为依据来决定是否要执行某些配置方法。

@ConditionalOnMissingBean表示只有不存在对应的类的`bean`才会自动配置该类。



#### @ConditionalOnClass

表示在类路径中存在类才会配置该配置类。只有引入相关依赖才会自动配置该配置类。



#### @ConditionalOnMissingClass

这两个注解属于类条件注解，它们根据是否存在某个类作为判断依据来决定是否要执行某些配置。



#### @ConditionalExpression

此注解可以让我们控制更细粒度的基于表达式的配置条件限制。当表达式满足某个条件或者表达式为真的时候，将会执行被此注解标注的方法。



#### @ConditionalOnProperty

@ConditionalOnProperty 注解会根据Spring配置文件中的配置项是否满足配置要求，从而决定是否要执行被其标注的方法。



```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
@Documented
@Conditional({OnPropertyCondition.class})
public @interface ConditionalOnProperty {
    //数组，获取对应property名称的值，与name不可同时使用
    String[] value() default {}; 
    //property名称的前缀，可有可无
    String prefix() default "";
 	//数组，property完整名称或部分名称（可与prefix组合使用，组成完整的property名称），与value不可同时使用
    String[] name() default {};
 	//可与name组合使用，比较获取到的属性值与havingValue给定的值是否相同，相同才加载配置
    String havingValue() default "";
 	//缺少该property时是否可以加载。如果为true，没有该property也会正常加载；反之报错
    boolean matchIfMissing() default false;
 	//是否可以松散匹配，至今不知道怎么使用的
    boolean relaxedNames() default true;
}
```



***example***

```java
@Bean
@ConditionalOnProperty(value = "sample.zipkin.enabled", havingValue = "false")
public Reporter<Span> spanReporter() {
    return Reporter.CONSOLE;
}
```



#### @ConditionalOnResource

此注解用于检测当某个配置文件存在使，则触发被其标注的方法。



#### @ConditionalOnWebApplication

#### @ConditionalOnNotWebApplication

这两个注解用于判断当前的应用程序是否是Web应用程序。如果当前应用是Web应用程序，则使用Spring WebApplicationContext,并定义其会话的生命周期。



## Configuration

### Core Configuration File

spring boot 核心的两个配置文件：

- bootstrap (. yml 或者 . properties)：boostrap 由父 ApplicationContext 加载的，比 applicaton 优先加载，配置在应用程序上下文的引导阶段生效。一般来说我们在 Spring Cloud Config 或者 Nacos 中会用到它。且 boostrap 里面的属性不能被覆盖；
- application (. yml 或者 . properties)： 由ApplicatonContext 加载，用于 spring boot 项目的自动化配置。



### Configuration Load Type

在 Spring Boot 里面，可以使用以下几种方式来加载配置。

- properties文件；
- YAML文件；
- 系统环境变量；
- 命令行参数；
- XML配置；



可以在 Spring Beans 里面直接使用这些配置文件中加载的值，如：

1、使用 @Value 注解直接注入对应的值，这能获取到 Spring 中 Environment 的值；

2、使用 @ConfigurationProperties 注解把对应的值绑定到一个对象；

3、直接获取注入 Environment 进行获取；

配置属性的方式很多，Spring boot使用了一种独有的 PropertySource 可以很方便的覆盖属性的值。



#### YAML配置

YAML 是一种人类可读的数据序列化语言。它通常用于配置文件。与属性文件相比，如果我们想要在配置文件中添加复杂的属性，YAML 文件就更加结构化，而且更少混淆。可以看出 YAML 具有分层配置数据。

YAML 现在可以算是非常流行的一种配置文件格式了，无论是前端还是后端，都可以见到 YAML 配置。那么 YAML 配置和传统的 properties 配置相比到底有哪些优势呢？

1. 配置有序，在一些特殊的场景下，配置有序很关键
2. 支持数组，数组中的元素可以是基本数据类型也可以是对象
3. 简洁

相比 properties 配置文件，YAML 还有一个缺点，就是不支持 @PropertySource 注解导入自定义的 YAML 配置。



#### XML配置

Spring Boot 推荐使用 Java 配置而非 XML 配置，但是 Spring Boot 中也可以使用 XML 配置，通过 @ImportResource 注解可以引入一个 XML 配置。

```java
@ImportResource({"classpath:some-context.xml","classpath:another-context.xml"})
```



### Configuration Load Order

1、开发者工具 `Devtools` 全局配置参数；

2、单元测试上的 [`@TestPropertySource](mailto:`@TestPropertySource)` 注解指定的参数；

3、单元测试上的 [`@SpringBootTest](mailto:`@SpringBootTest)` 注解指定的参数；

4、命令行指定的参数，如 `java -jar springboot.jar --name="Java技术栈"`；

5、命令行中的 `SPRING_APPLICATION_JSONJSON` 指定参数, 如 `java -Dspring.application.json='{"name":"Java技术栈"}' -jar springboot.jar`

6、`ServletConfig` 初始化参数；

7、`ServletContext` 初始化参数；

8、JNDI参数（如 `java:comp/env/spring.application.json`）；

9、Java系统参数（来源：`System.getProperties()`）；

10、操作系统环境变量参数；

11、`RandomValuePropertySource` 随机数，仅匹配：`ramdom.*`；

12、JAR包外面的配置文件参数（`application-{profile}.properties（YAML）`）

13、JAR包里面的配置文件参数（`application-{profile}.properties（YAML）`）

14、JAR包外面的配置文件参数（`application.properties（YAML）`）

15、JAR包里面的配置文件参数（`application.properties（YAML）`）

16、[`@Configuration](mailto:`@Configuration)`配置文件上 [`@PropertySource](mailto:`@PropertySource)` 注解加载的参数；

17、默认参数（通过 `SpringApplication.setDefaultProperties` 指定）；



数字小的优先级越高，即数字小的会覆盖数字大的参数值，我们来实践下，验证以上配置参数的加载顺序。



## Start Principle

### Start Process

![SpingBoot启动流程图](https://oss.xubighead.top/oss/image/202506/1930469466870747137.jpg)

### Auto Configuration

注解 @EnableAutoConfiguration, @Configuration, @ConditionalOnClass 就是自动配置的核心。

处理@Configuration的核心还是ConfigurationClassPostProcessor，这个类实现了BeanFactoryPostProcessor，因此当AbstractApplicationContext执行refresh方法里的invokeBeanFactoryPostProcessors(beanFactory)方法时会执行自动装配。



#### Related Annotation

##### @EnableAutoConfiguration

@EnableAutoConfiguration：用于根据用户所引用的jar包自动装配Spring容器，比如用户在ClassPath中包含了HSQLDB，但是没有手动配置数据库连接，那么Spring会自动使用HSQLDB作为数据源。

@EnableAutoConfiguration的功能可总结为：使Spring启用factories机制导入各个starter模块的配置。



Spring的自动装配发展大致可以分为三个阶段：

1. 全手工配置的XML文件阶段，用户需要的Bean全部需要在XML文件中声明，用户手工管理全部的Bean。
2. 半手工配置的注解阶段，用户可以安装需求Enable对应的功能模块，如添加@EnableWebMvc可以启用MVC功能。
3. 全自动配置的SpringBoot，用户只需要引入对应的starter包，Spring会通过factories机制自动装配需要的模块。



全手工配置的XML文件示意图：

![xml手工装配](https://oss.xubighead.top/oss/image/202506/1930469539499315202.jpg)

半自动注解配置示意图：

![半自动配置](https://oss.xubighead.top/oss/image/202506/1930469561641046018.jpg)

全自动注解配置示意图：

![全自动配置](https://oss.xubighead.top/oss/image/202506/1930469581022924802.jpg)

Spring启用全自动配置功能的注解就是@EnableAutoConfiguration，应用添加了@EnableAutoConfiguration注解之后，会读取所有jar包下面的spring.factories文件，获取文件中配置的自动装配模块，然后去装配对应的模块。



##### @Condition

@Condition：不同情况下按照条件进行装配，Spring的JdbcTemplate是不是在Classpath里面？如果是，并且DataSource也存在，就自动配置一个JdbcTemplate的Bean



##### @ComponentScan

@ComponentScan：扫描指定包下面的@Component注解的组件。

*如*`SpringBootApplication`注解源码所示，容器中将排除TypeExcludeFilterh和AutoConfigurationExcludeFilter。

```java
// ...
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
    // ...
}
```



#### Related Class

##### SpringFactoriesLoader

SpringFactoriesLoader的作用是从classpath/META-INF/spring.factories文件中，根据key来加载对应的类到spring IoC容器中。



##### AutoConfigurationPackages

对于标注@AutoConfigurationPackage注解的类的包，应当使用*`AutoConfigurationPackages`*注册。实质上，它负责保存标注相关注解的类的所在包路径。使用一个BasePackage类，保存这个路径。然后使用@Import注解将其注入到ioc容器中。这样，可以在容器中拿到该路径。



#### Source Code Analysis

@EnableAutoConfiguration 给容器导入META-INF/spring.factories 里定义的自动配置类。筛选有效的自动配置类。每一个自动配置类结合对应的 xxxProperties.java 读取配置文件进行自动配置功能。



在启动类上面有一个 `@SpringBootApplication` 注解，点进去可以看到如下内容：

> org.springframework.boot.autoconfigure.SpringBootApplication

```java
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
// ...
public @interface SpringBootApplication {
    // ...
}
```



在这个注解中，其中有一个 `@EnableAutoConfiguration` 注解，正是因为有了这样一个注解才得以实现自动装配的功能。继续往下面看。

> org.springframework.boot.autoconfigure.EnableAutoConfiguration

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
	// ...
}
```



- @AutoConfigurationPackage：将添加该注解的类所在的package作为自动配置package进行管理。*表示对于标注该注解的类的包，应当使用*`AutoConfigurationPackages`*注册。实质上，它负责保存标注相关注解的类的所在包路径。使用一个BasePackage类，保存这个路径。然后使用@Import注解将其注入到ioc容器中。这样，可以在容器中拿到该路径。*
- @Import({AutoConfigurationImportSelector.class})：用于导入factories文件中的AutoConfiguration。



##### Get package

查看*`AutoConfigurationPackages`*中的*`Registrar`*这个类的源码，在*`Registrar#registerBeanDefinitions`*方法中有这样一句代码*`new PackageImport(metadata).getPackageName()`*，查看PackageImport的构造器后不难发现，这里获取的是*`StandardAnnotationMetadata`*这个实例所在的包名。

metadata: 实际上是 StandardAnnotationMetadata 实例。 metadata#getClassName(): 获取标注 **@AutoConfigurationPackage** 注解的类的全限定名。ClassUtils.getPackageName(…): 获取其所在包。 



> org.springframework.boot.autoconfigure.AutoConfigurationPackages.Registrar

```java
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {

    @Override
    public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
        register(registry, new PackageImport(metadata).getPackageName());
    }
}
```



> org.springframework.boot.autoconfigure.AutoConfigurationPackages

```java
public static void register(BeanDefinitionRegistry registry, String... packageNames) {
    // BEAN：AutoConfigurationPackages类的全限定名
	// 此时判断BeanDefinitionRegistry中是否存在以BEAN作为beanName的BeanDefinition对象
	// 如果不存在，走else方法，构造了一个BackPackages实例，进行注册
    if (registry.containsBeanDefinition(BEAN)) {
        BeanDefinition beanDefinition = registry.getBeanDefinition(BEAN);
        ConstructorArgumentValues constructorArguments = beanDefinition.getConstructorArgumentValues();
        constructorArguments.addIndexedArgumentValue(0, addBasePackages(constructorArguments, packageNames));
    }
    else {
        GenericBeanDefinition beanDefinition = new GenericBeanDefinition();
        beanDefinition.setBeanClass(BasePackages.class);
        beanDefinition.getConstructorArgumentValues().addIndexedArgumentValue(0, packageNames);
        beanDefinition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
        registry.registerBeanDefinition(BEAN, beanDefinition);
    }
}
```



##### Import Selector

可以看到 `@EnableAutoConfiguration` 注解中有一个 `@Import({AutoConfigurationImportSelector.class})`，导入了一个 `AutoConfigurationImportSelector` 类，该类间接实现了 `ImportSelector` 接口，会按照注解内容去装载需要的Bean。实现了一个 `String[] selectImports(AnnotationMetadata importingClassMetadata);` 方法，这个方法的返回值是一个字符串数组，对应的是一系列主要注入到 `Spring IoC` 容器中的类名。当在 `@Import` 中导入一个 `ImportSelector` 的实现类之后，会把该实现类中返回的 `Class` 名称都装载到 `IoC` 容器中。



> org.springframework.boot.autoconfigure.AutoConfigurationImportSelector

```java
@Override
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    if (!isEnabled(annotationMetadata)) {
        return NO_IMPORTS;
    }
    // 获取需要自动装配的AutoConfiguration列表
    AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
        .loadMetadata(this.beanClassLoader);
    AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(autoConfigurationMetadata,
                                                                              annotationMetadata);
    // 获取自动装配类的类名称列表
    return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
}
```



一旦被装载到 `IoC` 容器中过后，在后续就可以通过 `@Autowired` 来进行使用了。接下来我们看下 `selectImports` 方法里面的实现，当中引用了 `getCandidateConfigurations` 方法 ，其中的 `ImportCandidates.load` 方法我们可以看到是通过加载 `String location = String.format("META-INF/spring/%s.imports", annotation.getName());` 对应路径下的 `org.springframework.boot.autoconfigure.AutoConfiguration.imports` 文件，其中就包含了很多自动装配的配置类。



> org.springframework.boot.autoconfigure.AutoConfigurationImportSelector

```java
protected AutoConfigurationEntry getAutoConfigurationEntry(AutoConfigurationMetadata autoConfigurationMetadata,
                                                           AnnotationMetadata annotationMetadata) {
    if (!isEnabled(annotationMetadata)) {
        return EMPTY_ENTRY;
    }
    // 获取注解中的属性
    AnnotationAttributes attributes = getAttributes(annotationMetadata);
    // 获取所有META-INF/spring.factories中的AutoConfiguration类
    List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
    // 删除重复的类
    configurations = removeDuplicates(configurations);
    // 获取注解中Execlud的类
    Set<String> exclusions = getExclusions(annotationMetadata, attributes);
    checkExcludedClasses(configurations, exclusions);
    // 移除所有被Exclude的类
    configurations.removeAll(exclusions);
    // 使用META-INF/spring.factories中配置的过滤器，过滤掉不需要使用的类
    configurations = filter(configurations, autoConfigurationMetadata);
    // 将自动配置的类，导入事件监听器，并触发fireAutoConfigurationImportEvents事件
	// 加载META-INF\spring.factories中的AutoConfigurationImportListener 
    fireAutoConfigurationImportEvents(configurations, exclusions);
    // 创建AutoConfigurationEntry对象
    return new AutoConfigurationEntry(configurations, exclusions);
}
```



##### Get Auto Configuration

getCandidateConfigurations方法获取候选配置信息，加载的是当前项目的classpath目录下的所有的 spring.factories 文件中的 key 为org.springframework.boot.autoconfigure.EnableAutoConfiguration 的信息。



> org.springframework.boot.autoconfigure.AutoConfigurationImportSelector

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
    List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
                                                                         getBeanClassLoader());
    Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
                    + "are using a custom packaging, make sure that file is correct.");
    return configurations;
}
```



> org.springframework.core.io.support.SpringFactoriesLoader

```java
public static List<String> loadFactoryNames(Class<?> factoryClass, @Nullable ClassLoader classLoader) {
    String factoryClassName = factoryClass.getName();
    return loadSpringFactories(classLoader).getOrDefault(factoryClassName, Collections.emptyList());
}

private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
    MultiValueMap<String, String> result = cache.get(classLoader);
    if (result != null) {
        return result;
    }

    try {
        // FACTORIES_RESOURCE_LOCATION值为"META-INF/spring.factories"
        Enumeration<URL> urls = (classLoader != null ?
                                 classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
                                 ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
        result = new LinkedMultiValueMap<>();
        while (urls.hasMoreElements()) {
            URL url = urls.nextElement();
            UrlResource resource = new UrlResource(url);
            Properties properties = PropertiesLoaderUtils.loadProperties(resource);
            for (Map.Entry<?, ?> entry : properties.entrySet()) {
                String factoryClassName = ((String) entry.getKey()).trim();
                for (String factoryName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
                    result.add(factoryClassName, factoryName.trim());
                }
            }
        }
        cache.put(classLoader, result);
        return result;
    }
    catch (IOException ex) {
        throw new IllegalArgumentException("Unable to load factories from location [" +
                                           FACTORIES_RESOURCE_LOCATION + "]", ex);
    }
}
```



从spring.factories文件中加载需要自动配置的类，可以看到这个文件中有一个 `RedisAutoConfiguration` 配置类，在这个配置中就有我们需要的 `RedisTemplate` 类的 `Bean`，同时也可以看到，在类上面有一个 `@ConditionalOnClass({RedisOperations.class})` 注解，表示只要在类路径上有 `RedisOperations.class `这个类的时候才会进行实例化。这也就是为什么只要添加了依赖，就可以自动装配的原因。

> org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration

```java
@Configuration
@ConditionalOnClass(RedisOperations.class)
@EnableConfigurationProperties(RedisProperties.class)
@Import({ LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class })
public class RedisAutoConfiguration {
    // ...
}
```



自动配置主要由`@EnableAutoConfiguration`实现，添加了`@EnableAutoConfiguration`注解，会导入`AutoConfigurationImportSelector`类，里面的`selectImports`方法通过`SpringFactoriesLoader.loadFactoryNames()`扫描所有含有`META-INF/spring.factories`的`jar`包,将对应`key`为`@EnableAutoConfiguration`注解全名对应的`value`类全部装配到`IOC`容器中。



##### Filter Mismatch Configuration

过滤，检查候选配置类上的注解@ConditionalOnClass，如果要求的类不存在，则这个候选类会被过滤不被加载。



> org.springframework.boot.autoconfigure.AutoConfigurationImportSelector#filter

```java
private List<String> filter(List<String> configurations, AutoConfigurationMetadata autoConfigurationMetadata) {
    long startTime = System.nanoTime();
    String[] candidates = StringUtils.toStringArray(configurations);
    boolean[] skip = new boolean[candidates.length];
    boolean skipped = false;
    for (AutoConfigurationImportFilter filter : getAutoConfigurationImportFilters()) {
        invokeAwareMethods(filter);
        boolean[] match = filter.match(candidates, autoConfigurationMetadata);
        for (int i = 0; i < match.length; i++) {
            if (!match[i]) {
                skip[i] = true;
                candidates[i] = null;
                skipped = true;
            }
        }
    }
    if (!skipped) {
        return configurations;
    }
    List<String> result = new ArrayList<>(candidates.length);
    for (int i = 0; i < candidates.length; i++) {
        if (!skip[i]) {
            result.add(candidates[i]);
        }
    }
    if (logger.isTraceEnabled()) {
        int numberFiltered = configurations.size() - result.size();
        logger.trace("Filtered " + numberFiltered + " auto configuration class in "
                     + TimeUnit.NANOSECONDS.toMillis(System.nanoTime() - startTime) + " ms");
    }
    return new ArrayList<>(result);
}
```



autoConfigurationMetadata参数是从META-INF/spring-autoconfigure-metadata.properties文件中解析出来的。

> org.springframework.boot.autoconfigure.AutoConfigurationImportSelector

```java
@Override
public String[] selectImports(AnnotationMetadata annotationMetadata) {
	// ...
    AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
        .loadMetadata(this.beanClassLoader);
    // ...
}
```



> org.springframework.boot.autoconfigure.AutoConfigurationMetadataLoader

```java
protected static final String PATH = "META-INF/" + "spring-autoconfigure-metadata.properties";
public static AutoConfigurationMetadata loadMetadata(ClassLoader classLoader) {
    return loadMetadata(classLoader, PATH);
}
```



#### Application Example

在第三方包的resources下新建文件夹META-INF，在文件夹下面新建spring.factories文件，文件中配置，key为自定配置类EnableAutoConfiguration的全路径，value是配置类的全路径

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.core.base.config.MyAutoConfiguration
```



创建对应的类：

```java
@Configuration
public class MyAutoConfiguration {
    @Bean
    public AutoConfigBean autoConfigBean(){
        return new AutoConfigBean();
    }
}
```



```java
@Slf4j
public class AutoConfigBean {
    public AutoConfigBean() {
        log.info("init auto config bean");
    }
}
```



重新打包，重新运行SpringBoot启动类。编写的MyAutoConfiguration类，就被加载进来了。



#### Summary

Spring Boot自动配置原理

- 1、`@EnableAutoConfiguration`注解导入`AutoConfigurationImportSelector`类。
- 2、执行`selectImports`方法调用`SpringFactoriesLoader.loadFactoryNames()`扫描所有`jar`下面的对应的`META-INF/spring.factories`文件.
- 3、限定为`@EnableAutoConfiguration`对应的`value`，将这些装配条件的装配到`IOC`容器中。

自动装配简单来说就是自动将第三方的组件的`bean`装载到`IOC`容器内，不需要再去写`bean`相关的配置，符合**约定大于配置**理念。

`Spring Boot`基于**约定大于配置**的理念，配置如果没有额外的配置的话，就给按照默认的配置使用约定的默认值，按照约定配置到`IOC`容器中，无需开发人员手动添加配置，加快开发效率。



### Start Extension Point

#### ApplicationContextInitializer

> org.springframework.context.ApplicationContextInitializer

```java
public interface ApplicationContextInitializer<C extends ConfigurableApplicationContext> {
	void initialize(C applicationContext);
}
```



整个spring容器在刷新之前初始化`ConfigurableApplicationContext`的回调接口，简单来说，就是在容器刷新之前调用此类的`initialize`方法。这个点允许被用户自己扩展。用户可以在整个spring容器还没被初始化之前做一些事情。

这时候spring容器还没被初始化，所以想要自己的扩展的生效，有以下三种方式：

- 在启动类中用`springApplication.addInitializers(new TestApplicationContextInitializer())`语句加入
- 配置文件配置`context.initializer.classes=com.example.demo.TestApplicationContextInitializer`
- Spring SPI扩展，在spring.factories中加入`org.springframework.context.ApplicationContextInitializer=com.example.demo.TestApplicationContextInitializer`



#### BeanDefinitionRegistryPostProcessor

> org.springframework.beans.factory.support.BeanDefinitionRegistryPostProcessor

```java
public interface BeanDefinitionRegistryPostProcessor extends BeanFactoryPostProcessor {
	void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException;
}
```



这个接口在读取项目中的`beanDefinition`之后执行，提供一个补充的扩展点。在这里动态注册自己的`beanDefinition`，可以加载classpath之外的bean。



#### BeanFactoryPostProcessor

> org.springframework.beans.factory.config.BeanFactoryPostProcessor

```java
@FunctionalInterface
public interface BeanFactoryPostProcessor {
	void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;
}
```



这个接口是`beanFactory`的扩展接口，调用时机在spring在读取`beanDefinition`信息之后，实例化bean之前。

可以通过实现这个扩展接口来自行处理一些东西，比如修改已经注册的`beanDefinition`的元信息。



#### InstantiationAwareBeanPostProcessor

> org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessor

```java
public interface InstantiationAwareBeanPostProcessor extends BeanPostProcessor {
	@Nullable
	default Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
		return null;
	}

	default boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
		return true;
	}

	@Nullable
	default PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName)
			throws BeansException {

		return null;
	}

	@Deprecated
	@Nullable
	default PropertyValues postProcessPropertyValues(
			PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) throws BeansException {

		return pvs;
	}

}
```



该接口继承了`BeanPostProcess`接口，区别在于**`BeanPostProcess`接口只在bean的初始化阶段进行扩展（注入spring上下文前后），而`InstantiationAwareBeanPostProcessor`接口在此基础上增加了3个方法，把可扩展的范围增加了实例化阶段和属性注入阶段。**

该类主要的扩展点有以下5个方法，主要在bean生命周期的两大阶段：**「实例化阶段」** 和**「初始化阶段」** ，下面一起进行说明，按调用顺序为：

- `postProcessBeforeInstantiation`：实例化bean之前，相当于new这个bean之前
- `postProcessAfterInstantiation`：实例化bean之后，相当于new这个bean之后
- `postProcessPropertyValues`：bean已经实例化完成，在属性注入时阶段触发，`@Autowired`,`@Resource`等注解原理基于此方法实现
- `postProcessBeforeInitialization`：初始化bean之前，相当于把bean注入spring上下文之前
- `postProcessAfterInitialization`：初始化bean之后，相当于把bean注入spring上下文之后



这个扩展点非常有用 ，无论是写中间件和业务中，都能利用这个特性。比如对实现了某一类接口的bean在各个生命期间进行收集，或者对某个类型的bean进行统一的设值等等。



#### SmartInstantiationAwareBeanPostProcessor

> org.springframework.beans.factory.config.SmartInstantiationAwareBeanPostProcessor

```java
public interface SmartInstantiationAwareBeanPostProcessor extends InstantiationAwareBeanPostProcessor {
	@Nullable
	default Class<?> predictBeanType(Class<?> beanClass, String beanName) throws BeansException {
		return null;
	}

	@Nullable
	default Constructor<?>[] determineCandidateConstructors(Class<?> beanClass, String beanName)
			throws BeansException {

		return null;
	}

	default Object getEarlyBeanReference(Object bean, String beanName) throws BeansException {
		return bean;
	}
}
```



该扩展接口有3个触发点方法：

- `predictBeanType`：该触发点发生在`postProcessBeforeInstantiation`之前(在图上并没有标明，因为一般不太需要扩展这个点)，这个方法用于预测Bean的类型，返回第一个预测成功的Class类型，如果不能预测返回null；当你调用`BeanFactory.getType(name)`时当通过bean的名字无法得到bean类型信息时就调用该回调方法来决定类型信息。
- `determineCandidateConstructors`：该触发点发生在`postProcessBeforeInstantiation`之后，用于确定该bean的构造函数之用，返回的是该bean的所有构造函数列表。用户可以扩展这个点，来自定义选择相应的构造器来实例化这个bean。
- `getEarlyBeanReference`：该触发点发生在`postProcessAfterInstantiation`之后，当有循环依赖的场景，当bean实例化好之后，为了防止有循环依赖，会提前暴露回调方法，用于bean实例化的后置处理。这个方法就是在提前暴露的回调方法中触发。



#### BeanFactoryAware

> org.springframework.beans.factory.BeanFactoryAware

```java
public interface BeanFactoryAware extends Aware {
	void setBeanFactory(BeanFactory beanFactory) throws BeansException;
}
```



这个类只有一个触发点，发生在bean的实例化之后，注入属性之前，也就是Setter之前。这个类的扩展点方法为`setBeanFactory`，可以拿到`BeanFactory`这个属性。

使用场景为，可以在bean实例化之后，但还未初始化之前，拿到 `BeanFactory`，在这个时候，可以对每个bean作特殊化的定制。也或者可以把`BeanFactory`拿到进行缓存，日后使用。



## Application Example

### Cross Origin

跨域可以在前端通过 JSONP 来解决，但是 JSONP 只可以发送 GET 请求，无法发送其他类型的请求，在 RESTful 风格的应用中，就显得非常鸡肋，因此我们推荐在后端通过 （CORS，Cross-origin resource sharing） 来解决跨域问题。这种解决方案并非 Spring Boot 特有的，在传统的 SSM 框架中，就可以通过 CORS 来解决跨域问题，只不过之前我们是在 XML 文件中配置 CORS ，现在可以通过实现WebMvcConfigurer接口然后重写addCorsMappings方法解决跨域问题。



#### 配置类方式

##### SpringMVC方式

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowCredentials(true)
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .maxAge(3600);
    }

}
```



##### 过滤器方式

一个http请求，先走filter，到达servlet后才进行拦截器的处理，如果我们把cors放在filter里，就可以优先于权限拦截器执行。

```java
@Configuration
public class CorsConfig {

    @Bean
    public CorsFilter corsFilter() {
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.addAllowedOrigin("*");
        corsConfiguration.addAllowedHeader("*");
        corsConfiguration.addAllowedMethod("*");
        corsConfiguration.setAllowCredentials(true);
        UrlBasedCorsConfigurationSource urlBasedCorsConfigurationSource = new UrlBasedCorsConfigurationSource();
        urlBasedCorsConfigurationSource.registerCorsConfiguration("/**", corsConfiguration);
        return new CorsFilter(urlBasedCorsConfigurationSource);
    }

}
```



### Global Exception Interception

Spring 提供了一种使用 ControllerAdvice 处理异常的非常有用的方法。 我们通过实现一个 ControlerAdvice 类，来处理控制器类抛出的所有异常。



# Spring_Boot_Starter

启动器是一套方便的依赖没描述符，它可以放在自己的程序中。你可以一站式的获取你所需要的 Spring 和相关技术，而不需要依赖描述符的通过示例代码搜索和复制黏贴的负载。

优点：

- 减少jar包的引入；
- 统一管理依赖之间的版本。



Spring Boot 也提供了其它的启动器项目包括，包括用于开发特定类型应用程序的典型依赖项。

- spring-boot-starter-web-services - SOAP Web Services

- spring-boot-starter-web - Web 和 RESTful 应用程序

- spring-boot-starter-test - 单元测试和集成测试

- spring-boot-starter-jdbc - 传统的 JDBC

- spring-boot-starter-hateoas - 为服务添加 HATEOAS 功能

- spring-boot-starter-security - 使用 SpringSecurity 进行身份验证和授权

- spring-boot-starter-data-jpa - 带有 Hibeernate 的 Spring Data JPA

- spring-boot-starter-data-rest - 使用 Spring Data REST 公布简单的 REST 服务



## spring-boot-starter-security

为了实现 Spring Boot 的安全性，我们使用 spring-boot-starter-security 依赖项，并且必须添加安全配置。它只需要很少的代码。配置类将必须扩展WebSecurityConfigurerAdapter 并覆盖其方法。



# Component_Integrate

## Mybatis

## Mybatis-plus




# References
- [Spring Boot 官方英文文档](https://docs.spring.io/spring-boot/docs/2.0.0.M3/reference/htmlsingle/)
- [SpringBoot中 Jackson 日期的时区和日期格式问题](https://blog.csdn.net/jianxia801/article/details/89741073)