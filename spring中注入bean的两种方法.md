目前在自己所接触的知识范围内，觉得在spring mvc中bean的配置应该有两种不同的方法。分别是基于XML的配置和基于Annotation的配置。所以，这里对此做一个小小的总结。

####基于XML的配置

基于XML的配置，这里其实又分为两种情况。一种是通过setter方法注册相应的bean; 另外一种是通过构造函数来注册（这里的情况都是在某一个bean里面需要用到其它的bean)。

##### 1.setter注入

在前一篇文章中已经介绍了`spring.xml`这个文件应该存放的正确位置，那么在setter注入中，xml文件相应的配置如下：

```xml
<bean id="addCalculator" class="com.tw.bean.operation.AddCalculator"/>
<bean id="plusCalculator" class="com.tw.bean.operation.PlusCalculator"/>
<bean id="calculatorClient" class="com.tw.bean.CalculatorClient">
    <property name="addCalculator" ref="addCalculator"></property>
    <property name="plusCalculator" ref="plusCalculator"></property>
</bean>
```

如果进行了以上的配置，那么在`CalculatorClient.java`这个文件下，就必须要有如下的方法：

```java
public void setAddCalculator(ICalculator addCalculator) {
    this.addCalculator = addCalculator;
}
public void setPlusCalculator(ICalculator plusCalculator) {
    this.plusCalculator = plusCalculator;
}
```

##### 2.构造注入

如果是构造注入，其实xml配置文件和之前的setter注入中xml的配置还是很类似的，相应的配置如下：

```xml
<bean id="addCalculator" class="com.tw.bean.operation.AddCalculator"></bean>
<bean id="plusCalculator" class="com.tw.bean.operation.PlusCalculator"></bean>
<bean id="calculatorClient" class="com.tw.bean.CalculatorClient">
    <constructor-arg index="0" ref="addCalculator"></constructor-arg>
    <constructor-arg index="1" ref="plusCalculator"></constructor-arg>
</bean>
```

既然是构造注入，那么在相应的`CalculatorClient.java`这个文件下，就必须要有相应的构造函数。如下：

```java
public CalculatorClient(ICalculator addCalculator, ICalculator plusCalculator) {
    this.addCalculator = addCalculator;
    this.plusCalculator = plusCalculator;
}
```

####基于annotation的配置

如果bean的注入是基于annotation配置，那么就需要用到一些注解。常用的注解如：`@Component`,`@Autowired`。当然仅仅这些还不够，还是需要一些相应的配置信息，这里简单介绍一下我所知道的几种配置。

#####1.xml配置

这里的xml配置其实要远远比上面的xml配置简单，因为在这种情况下bean的注入已经用到了相应的注解。所以，就不需要在xml配置文件中配置bean的注入。当然，在xml文件中可以用到下面的语句：

```xml
<context:component-scan base-package="com.tw.bean"/>
```

之后，在主函数上获取这个xml文件即可。

```java
ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
```

#####2.通过java class进行配置

如果通过一个`AppConfig.class`这个类进行配置，就不需要任何xml文件。在这种情况下，只需要新建一个类，如下所示：

```java
@Configuration
@ComponentScan("com.tw.bean")
public class AppConfig {
}
```

而在main函数中，只需要下面的语句即可

```java
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
```

#####3.直接调用
这种方法指的是在main函数中直接调用方法，既不需要用到xml的配置，也不需要用到AppConfig.class的配置，如下：

```java
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
context.scan("com.tw.bean");
context.refresh();
```

虽然这种方法看似很美好，但是如果项目后期功能越来越多，那么你就需要配置一些其他的参数，在这种情况下，这样的一种方法就显得很有局限性了。所以，个人觉得用xml或AppConfig配置会显得更好一点。

当然，前面提到过注解的运用如下所示：

```java
@Component
public class CalculatorClient {
    @Autowired
    private ICalculator addCalculator;

    @Autowired
    private ICalculator plusCalculator;
}
```