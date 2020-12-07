# Spring配置——注解配置

Spring有两大配置情况，首先是基于XML的配置，这是最传统的配置，能够实现分离开发，便于后期添加功能和修改。

但是，XML的配置有着一些比较难以克服的缺点：

- XML文件的维护使得一些额外的工作需要完成
- XML文件的代码与配置分离操作不利于代码的阅读
- XML文件难以实现自动化配置，大量依赖人工配置。

因此，为了克服以上缺点，Spring提供了另一种配置方案，也就是基于注解的配置，这也是当下Java程序发展的方向。而且在SpringBoot中，都是使用注解进行相关配置，这里通过SpringBoot的部分源码进行一下分析：

```java
@SpringBootApplication
public class SpringstudyApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringstudyApplication.class, args);
	}

}// 启动一个Spring容器


// SpringApplication.run
public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
		return run(new Class<?>[] { primarySource }, args);
}
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
	return new SpringApplication(primarySources).run(args);
}

public ConfigurableApplicationContext run(String... args) {
    // ......
	context = createApplicationContext();
    // ......
    return context;
}

// SpringApplication.createApplicationContext()
protected ConfigurableApplicationContext createApplicationContext() {
	return this.applicationContextFactory.create(this.webApplicationType);
}

// SpringApplication.applicationContextFactory
private ApplicationContextFactory applicationContextFactory = ApplicationContextFactory.DEFAULT;

// ApplicationContextFactory.DEFAULT
ApplicationContextFactory DEFAULT = (webApplicationType) -> {
		try {
			switch (webApplicationType) {
			case SERVLET:
				return new AnnotationConfigServletWebServerApplicationContext();
			case REACTIVE:
				return new AnnotationConfigReactiveWebServerApplicationContext();
			default:
				return new AnnotationConfigApplicationContext();
			}
		}
		catch (Exception ex) {
			throw new IllegalStateException("Unable create a default ApplicationContext instance, "
					+ "you may need a custom ApplicationContextFactory", ex);
		}
	};
```

以上可以看出，能够启动的容器就是处理注解相关的容器。

因此需要首先对SpringFramework中的相关注解进行学习。

## @Component

这是配置bean的注解中最高一层的注解了，是@Configuration的父注解。

其主要用法和功能：**用该注解修饰一个类，可以将一个类注册成容器中的bean**。

```java
@Component("ID")
public class UserDaoImpl implements UserDao
{
    @Override
    public void save() {
        System.out.println("存储成功");
    }
}
```

上面代码将一个UserDaoImpl类注册成了一个容器中的bean，类似于XML文件中在`<beans>`中注册`<bean>`

这里需要注意和@Component作用完全一致的三个不同注解

- @Controller（控制层代码）
- @Service（服务层代码）
- @Repository（数据访问层代码）

上面三个注释的作用和@Component作用完全一样，只是更好的标注和区分了不同的层级，做到更好的开发。



## @Resource、@Value

用在@Component及其子注解修饰的类中

这两个配置用来给bean配置注入，分别用于配置bean的ref和value属性，它们既可以修饰set方法也可以直接修饰成员变量。

```java
@Component("ID")
public class UserDaoImpl implements UserDao
{
    @Value("view")
    private String name;
    private User user;
    
    @Resource("manager")
    public void setUser(User user) {
        this.user = user;
    }
    
    @Override
    public void save() {
        System.out.println("存储成功");
    }
}
```

注解中配置的值都是要注入的值，@Value中的是直接注入的值，而@Resource则是另一个bean的名字，而在@Resource后面不接名字时，按修饰的名字取查找bean。



## @PostConstruct、@PreDestroy

用在@Component及其子注解修饰的类中

这两个用于标注配置的bean的init-method和destroy-method方法。



## @DependsOn、@Lazy

用于任意被@Component及其子注解修饰的类上

@DependsOn用于在被修饰的bean初始化之前强制初始化别的bean，它可以接受一个字符串数组参数，数组中的每一项都是一个bean的名字。

@Lazy用于取消修饰的Bean的预初始化行为。



## @Scope、@Lookup

前者用于任意被@Component及其子注解修饰的类上，后者由于lookup方法的特殊性只能用于任意被@Component及其子注解修饰的抽象类上。

@Scope用于指定被修饰的bean的生命周期，可以直接接受一个生命周期的值。

@Lookup用于指定一个lookup方法，解决singleton作用域bean需要prototype作用域bean的注入问题。



## @Autowired

用于任意被@Component及其子注解修饰的类中的set方法、普通方法、成员变量、构造器。

该注解用于自动装配，而其默认搜索方式时“byType”也就是根据被修饰的成员的注入类型来查找，如果找到有很多同种类型的bean，则直接报错。

下面用@Autowired修饰成员变量来展示一些特殊用法：

```java
@Autowired
private Axe[] axes;
```

当@Autowired修饰一个数组时，它会自动找寻整个容器中该类型的bean并将其注入到数组当中。

同时对于任意的泛型容器，@Autowired也是采取和数组同样的措施进行注入：

```java
@Autowired
private List<Axe> axes;
```



接着，当@Autowired进行检索时，经常会检索出一些类型相同的bean，这样会导致容器报错，此时可以使用`@Primary`注解修饰那些类中的其中一个，此时就会默认注入那一个被修饰的bean。



## @Configuration、@Bean

这是非常重要的两个注解，实现了`<beans>`和`<bean>`的配置。

@Configuration是@Component的子注释，二者有着几乎差不多的功能，但是@Configuration更像是注册一个`<beans>`标签，Spring容器也会为其生成CGLib代理，以至于在被修饰的类中方法互相访问是无需重新创建对象，而@Component则不行，内部方法互相访问会重新创建被修饰类的对象。

@Bean则是在@Configuration及其子注释修饰的类中用于注册一个bean的注释，该注释注册的bean更多的集中在其生成的返回值的逻辑上，能通过这个逻辑代码进行bean的配置。

二者还需要进行一次详细的介绍。





## @ComponentScan

该注解修饰的bean被注册后将会自动扫描其包下所有的bean



## AnnotationConfigApplicationContext

该类用于注册一个处理注解的Spring容器：

```java
public class JavaConfigTest {
    public static void main(String[] arg) {
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
        ctx.register(AppConfig.class); // AppConfig可以带上一个@ComponentScan注解
        ctx.refresh();

        Entitlement ent = (Entitlement)ctx.getBean("entitlement");
        System.out.println(ent.getName());
        System.out.println(ent.getTime());

        Entitlement ent2 = (Entitlement)ctx.getBean("entitlement2");
        System.out.println(ent2.getName());
        System.out.println(ent2.getTime());

        ctx.close();
    }
```





