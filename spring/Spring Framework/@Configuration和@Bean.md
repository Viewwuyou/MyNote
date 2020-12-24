# 前言

在配置Spring的Bean的过程中，有XML和注解配置，早期的Spring注解配置只有以`@Component`为代表的一系列配置方式，这种配置方法将该注解修饰于一个类前面，直接将类注册为spring的注解。

但是这种方法必然有其局限性，如果我需要再额外的对注册的bean进行一些改造那么就不方便，因此基于Java的配置`@Configuration`和`@Bean`就成为了一个比较好的注解方法。



# @Configuration

该注解等价于XML配置中的`<beans>`标签，用于注册一个配置类。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Configuration {
	@AliasFor(annotation = Component.class)
	String value() default "";
	boolean proxyBeanMethods() default true;

}
```

根据源代码可以看出来，`@Configuration`本质也是一个`@Component`，因此本身就会被注册成为一个bean，只是该bean不怎么会被访问到，只是作为一个配置类来用。



# @Bean

该配置等价于XML配置中的`<bean>`标签,其返回值注册为一个SpringBean

```java
@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Bean {
	@AliasFor("name")
	String[] value() default {};
	@AliasFor("value")
	String[] name() default {};
	@Deprecated
	Autowire autowire() default Autowire.NO;
	boolean autowireCandidate() default true;
	String initMethod() default "";
	String destroyMethod() default AbstractBeanDefinition.INFER_METHOD;

}
```

该注解可以通过initMethod和destroyMethod两个属性配置初始化和销毁前的方法，并不能通过该注解配置作用域，如果需要配置作用域，需要使用`@Scope`注解进行配置。

