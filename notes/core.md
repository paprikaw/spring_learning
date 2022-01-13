# 核心

## Why spring
1. 依赖注入


## Core container

## AOP

## Data access layer

## Spring projects

下载spring jar包：https://repo.spring.io/ui/packages

## Ioc(Inversion of control)
将对程序的控制从程序员手里交还给使用者。我们可以在某种程度上config一个程序

## Spring开发流程


1. 配置spring bean 

2. 创建一个spring container 
  配置spring container有三种方式
   1. xml
   2. annotation
   3. java sourcecode
  spring container实际上是一个applicationContext

3. 从Spring container获取spring bean

    ![picture 3](images/cf88a48abad5a65cfed11536e5341d06ea658ad51e9d5a8682cd20469c5f4094.png)  

上图所示这个例子中，`getBean` 取一个id为`myCoach`的bean，implementation为`Coach.class`

## Dependency inject
一个bean可以依赖于多个beans，这些依赖被正确定义之后spring container就可以帮我们自动组装了

两种依赖注入的方式
* Constructor injection
    1. Define the dependency interface and class

        ![picture 5](images/b1df1eae6e1f6de917b333a1cc071eca09986a5caee583540feb5aeeb4e2034f.png)  
    2. Create a constructor in your class for injections

        ![picture 6](images/9c815fd80ca84e32d73c5e7b16e90d755b17d874f2c68e9d5d44f260afe9198c.png)  
    3. Configure the dependency inje ction in Spring config file

        ![picture 7](images/7d671caf2cc4982890d83cf2bcfc8baccf88c66a972c7fbd8a0916b9ec4948c0.png)

    思考：
    Constructor injection实际上就像是spring container帮我们向我们的对象中传参，传的参就是对象中的依赖。这就想一个函数一样，变量没有被写死，整个函数就有了更多的通用性。
    
* Setter Injection
  ![picture 11](images/fd8a9aaa01fb7dcd882ed4aa935549080ebbd35d81e14dbded292305c048048e.png)  

  ![picture 10](images/ef854b44dc0378cd9d9bba1e70eb19529393a9ffdfb2926c1b8f672f1bcf3f56.png)  

## Inject literal values

1. Create setter method(s) in your class for injections

2. Configure the injection in Spring config file

## 从Property files中注入property

1. 定义一个property file
   ``` json
    //sport.properties
    foo.email=myeasycoach@luv2code.com
    foo.team=Royal Challengers Bangalore
   ```
2. 使用 在`applicationContext.xml` 中定义property placeholder
   ``` xml
   	<!-- load the properties file: sport.properties -->
    <context:property-placeholder location="classpath:sport.properties"/>
   ```
3. 使用 在`applicationContext.xml` 中使用property placeholder
   ``` xml
    <bean id="myCricketCoach"
    		class="com.luv2code.springdemo.CricketCoach">

		<!-- set up setter injection -->
		<property name="fortuneService" ref="myFortuneService" /> 
		
		<!-- inject literal values -->   
		<property name="emailAddress" value="${foo.email}" />
		<property name="team" value="${foo.team}" />
				
    </bean>
   ```

## Bean scope
### Singleton
Spring container只为一个bean创建一个实例，然后所有对这个bean的引用都share一个memory。这种方式适合stateless的bean,如下图所示

![picture 12](images/c7d28834264c088157e2c6a4d447002a3a70ff108aa0701b85143c62485fdec1.png)  


还有各种不同的scope

![picture 13](images/5db35271f79e7c0026591770321ae4cd54fa68a227d3408280388291170d56be.png)  

### Prototype scope

spring container为每一个reference创建新的实例

![picture 14](images/ebdfaadbc1c36e29174ab7248f34eeff3686d89722583efe1f1d65cad27c2e8f.png)  

## Bean lifecycle

在bean初始化和被销毁的时候，我们可以定义一些hooks来做一些操作

关于初始化或销毁的function，我们有一些规则

* When using XML configuration, I want to provide additional details regarding the method signatures of the init-method  and destroy-method .

* Access modifier

    The method can have any access modifier (public, protected, private)

* Return type

    The method can have any return type. However, "void' is most commonly used. If you give a return type just note that you will not be able to capture the return value. As a result, "void" is commonly used.

* Method name

    The method can have any method name.

* Arguments

    The method can not accept any arguments. The method should be no-arg.

定义初始化或销毁函数的过程:
1. 在bean内定义初始化或销毁函数
    ``` java
    // add an init method
	public void doMyStartupStuff() {
		System.out.println("TrackCoach: inside method doMyStartupStuff");
	}
	
	// add a destroy method
	public void doMyCleanupStuffYoYo() {
		System.out.println("TrackCoach: inside method doMyCleanupStuffYoYo");		
	}
    ```
2. 在xml文件里面的bean中定义这些函数

    ``` xml
 	<bean id="myCoach"
 		class="com.luv2code.springdemo.TrackCoach"
 		init-method="doMyStartupStuff"
 		destroy-method="doMyCleanupStuffYoYo">	
 		
 		<!-- set up constructor injection -->
 		<constructor-arg ref="myFortuneService" />
 	</bean>
    ```
*注意：*

    For "prototype" scoped beans, Spring does not call the destroy method. 
具体参考[这里](../FAQ/Section%2006%20-%20Spring%20Bean%20Scopes%20and%20Lifecycle/049-special-note-about-destroy-lifecycle-and-prototype-scope.pdf)


## Anotation
使用spring注解来配置的步骤：
1. 在config file中enable spring annotation
    ``` xml
    	<context:component-scan base-package="com.luv2code.springdemo" />
    ```

2. 使用@container注解来注册bean
    ``` java
        @Component("thatSillyCoach")
    public class TennisCoach implement
    	@Override
    	public String getDailyWorkout() {
    		return "Practice your backhand volley";
        }
    }
    ```
3. 从spring container中retrieve bean

## 默认的bean ID
![picture 2](../images/05123c1fce90d097ab3f073a17ddba1315fa06a946e49ce36c0500c734e70ad1.png)

如果不注明bean的Id的话，就用以上的规则生成默认的bean ID

## Autowiring

### autowiring的步骤:
1. spring scan所有component注释的object
2. 在object中，如果有autowiring注释的，就寻找bean中有没有实现这个interface的bean，有的话就注入

### 三种注入方式

1. constructor 注入

   步骤：

   ![picture 3](../images/fcc96b5fd322a7c4eb8cec5d4bb22ed4e525dd70af5c69d4aa23b855f49f1503.png)  
 
    注意：如果一个object只有一个constructor，那么autowired是可选的。如果有多个constructor，那么必须注释autowired告诉spring哪一个constructor是用来DI的

2. setter 注入

    步骤和constructor一样，只不过改成了setter注入，在setter function上注释@autowired

3. method 注入
    
    和setter注入一样，只不过method 注入可以不止使用setter function，它可以使用任何function
4. Field 注入

    直接在filed上注释 @autowired，不需要constructor和setter


## 如果有多个interface，怎么办？

使用@Qualifier
``` java
@Autowired
@Qualifier("randomFortuneService")
private FortuneService fortuneService;
```
Qualifier中写bean id来定位一个特定的implementation

讲Qualifier写在constructor和setter中有一点不一样:

具体请看: [Constructor](../FAQ/Section%2008%20-%20Spring%20Configuration%20with%20Java%20Annotations%20-%20Dependency%20Injection/074-faq-how-to-inject-properties-file-using-java-annotations.pdf)


## 使用anotation注入property file

具体请看：[`property file注入方法`](../FAQ/Section%2008%20-%20Spring%20Configuration%20with%20Java%20Annotations%20-%20Dependency%20Injection/075-practice-activity-5-dependency-injection-with-annotations.pdf)


## Scope #Annotation

Scope annotation:
``` java
@Component
@Scope("prototype")
public class TennisCoach implements Coach {
    // ...
}
```

## @PostConstruct and @PreConstruct

在定义好的销毁或初始化函数上面加上这两个注释就行:
``` java
@Component
public class TennisCoach implements Coach {

	@Autowired
	@Qualifier("randomFortuneService")
	private FortuneService fortuneService;
	
	// define a default constructor
	public TennisCoach() {
		System.out.println(">> TennisCoach: inside default constructor");
	}

	// define my init method
	@PostConstruct
	public void doMyStartupStuff() {
		System.out.println(">> TennisCoach: inside of doMyStartupStuff()");
	}
	
	// define my destroy method
	@PreDestroy
	public void doMyCleanupStuff() {
		System.out.println(">> TennisCoach: inside of doMyCleanupStuff()");		
        //...other stuffs
	}
```

*注意*: 在java 9或以上中，无法使用PostConstruct和PreConstruct

[解决方法](../FAQ/Section/../Section%2009%20-%20Spring%20Configuration%20with%20Java%20Annotations%20-%20Bean%20Scopes%20and%20Lifecycle%20Methods/080-heads-up-for-java-9-users-postconstruct-and-predestroy.pdf)

## 使用java code来配置

步骤

![picture 1](../images/3e61ba7edc11396907b7907bb9821aa7c9c83883516f56ade181785ecabe041a.png)  

1. 创建一个java class，注释为configuration, 注释componentScan(扫描项目中有component注释的class)

    ``` java
    @Configuration
    @ComponentScan("com.luv2code.springdemo")
    public class SportConfig {
        
    }  
    ```

2. 在创建applicationContext的时候，使用`AnnotationConfigApplicationContext`
    ``` java
    // read spring config java class
    AnnotationConfigApplicationContext context = 
            new AnnotationConfigApplicationContext(SportConfig.class);
    ```

## 使用javacode来配置bean

步骤

1. 创建好class

``` java
public class SwimCoach implements Coach {

	private FortuneService fortuneService;

	public SwimCoach(FortuneService theFortuneService) {
		fortuneService = theFortuneService;
	}
	
	@Override
	public String getDailyWorkout() {
		return "Swim 1000 meters as a warm up.";
	}

	@Override
	public String getDailyFortune() {
		return fortuneService.getFortune();
	}

}
```
这里SwimCoach通过constructor来传dependency（和constructor DI一样）

2. 设置java code config

``` java
@Configuration
// @ComponentScan("com.luv2code.springdemo")
public class SportConfig {
	
	// define bean for our sad fortune service
	@Bean
	public FortuneService sadFortuneService() {
		return new SadFortuneService();
	}
	
	// define bean for our swim coach AND inject dependency
	@Bean
	public Coach swimCoach() {
		SwimCoach mySwimCoach = new SwimCoach(sadFortuneService());
		
		return mySwimCoach;
	}
	
}
```

具体的解析请看[这里](../FAQ/Section%2010%20-%20Spring%20Configuration%20with%20Java%20Code%20(no%20xml)/090-faq-how-bean-works-behind-the-scenes.pdf)

## 什么时候使用Bean?

一般的，我们会使用@component来注释bean。但是有的时候一些第三方的class没有使用spring，就会需要使用java code将这些class wrap成bean

## 使用java code config注入property file

步骤：
1. 创建一个property文件
2. 在config code上使用注解:

``` java
@Configuration
// @ComponentScan("com.luv2code.springdemo")
@PropertySource("classpath:sport.properties")
public class SportConfig {
    // ...KJ
}
```

3. 在目标class中使用注解注入相关的path

``` java
@Value("${foo.email}")
private String email;
	
@Value("${foo.team}")
private String team;
```
