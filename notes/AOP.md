# Aspect oriented programming


## 基本概念

通过配置将不同的服务直接插入到各种class的之前或者之后


![picture 18](/images/ee73da0ee179b9bc9488ea73055ce064ea60b8ee924fbadd07cbd3a2327bd58f.png)  


## Proxy design pattern

![picture 19](/images/80b1e7c8c7dcdc33bcbc9e13cb34dc370726e11bba28a01d981af85cf83e48ad.png)  

Main app不知道aop的存在，这是一个非常好的benifits


## AOP的使用场景
![picture 20](/images/8af4a96c14a486a9f0f45e92cb987d32da06e00c6c159b257988ce3ba1f574e0.png)  

## Pros and cons
![picture 21](/images/54f9a90e5f5d4885f4e46df311bf52f856f30be582a019c4277c7b6505901986.png)  

## Terminology

![picture 22](/images/3876ed7d0ade9346e30a8799eb47583e499bcf0c4fcb8666767a9a6dee2276e4.png)  


## Advice types

![picture 23](/images/c1801bb74baf90b7860a59248cfdc95ce619b88c1f25e38517bee72abf9004fe.png)  


# Before Advice

## 步骤

1. 加入AOPdependencies
2. 使用java code配置项目

    config file:
    ``` java
    package com.luv2code.aopdemo;

    import org.springframework.context.annotation.ComponentScan;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.context.annotation.EnableAspectJAutoProxy;

    @Configuration
    @EnableAspectJAutoProxy
    @ComponentScan("com.luv2code.aopdemo")
    public class DemoConfig {

    }
    ```

3. 编写DAO类

    AccountDao.java

    ``` java
    package com.luv2code.aopdemo.dao;

    import org.springframework.stereotype.Component;

    @Component
    public class AccountDAO {

    	public void addAccount() {
        
    		System.out.println(getClass() + ": DOING MY DB WORK: ADDING AN ACCOUNT");
    
    	}
    }

    ```
    
    在这个DAO中，获取数据知识简单的print一个string


4. 编写aspect class
    ``` java
    package com.luv2code.aopdemo.aspect;

    import org.aspectj.lang.annotation.Aspect;
    import org.aspectj.lang.annotation.Before;
    import org.springframework.stereotype.Component;

    @Aspect
    @Component
    public class MyDemoLoggingAspect {

    	// this is where we add all of our related advices for logging
    
    	// let's start with an @Before advice

    	@Before("execution(public void addAccount())")
    	public void beforeAddAccountAdvice() {
        
    		System.out.println("\n=====>>> Executing @Before advice on addAccount()");
    
    	}
    }

    ```

最后的src目录结构如下
```
.
└── com
    └── luv2code
        └── aopdemo
            ├── DemoConfig.java
            ├── MainDemoApp.java
            ├── aspect
            │   └── MyDemoLoggingAspect.java
            └── dao
                └── AccountDAO.java
```


# pointcut
    pointcut是一个断言语句，用来告诉AOP这个aspect能被apply在什么地方

## Pointcut expression

```
execution(modifiers-pattern? return-type-pattern declaring-type-pattern? method-name-pattern(param-pattern)throws-pattern?)
```


![picture 24](/images/cf04e019bf91ed3722589cb74759c70a3890ef006e34aacdabe1eebfdbffe395.png)  

## Reusing pointcut

``` java
@Aspect
@Component
public class MyDemoLoggingAspect {

	@Pointcut("execution(* com.luv2code.aopdemo.dao.*.*(..))")
	private void forDaoPackage() {}
	
	@Before("forDaoPackage()")
	public void beforeAddAccountAdvice() {		
		System.out.println("\n=====>>> Executing @Before advice on method");		
	}
	
	@Before("forDaoPackage()")
	public void performApiAnalytics() {
		System.out.println("\n=====>>> Performing API analytics");		
	}
	
}
```

声明了`@Pointcut`的方法可以被reuse

## Combine pointcut

使用logic operator来combine不同的pointcut

![picture 25](/images/641021468511723e14785f4814d9e889f193b9b1430a2e36aaac9de49e4a9718.png)  


例子：
``` java
	@Pointcut("execution(* com.luv2code.aopdemo.dao.*.*(..))")
	private void forDaoPackage() {}
	
	// create pointcut for getter methods
	@Pointcut("execution(* com.luv2code.aopdemo.dao.*.get*(..))")
	private void getter() {}
	
	// create pointcut for setter methods
	@Pointcut("execution(* com.luv2code.aopdemo.dao.*.set*(..))")
	private void setter() {}
	
	// create pointcut: include package ... exclude getter/setter
	@Pointcut("forDaoPackage() && !(getter() || setter())")
	private void forDaoPackageNoGetterSetter() {}
	
	@Before("forDaoPackageNoGetterSetter()")
	public void beforeAddAccountAdvice() {		
		System.out.println("\n=====>>> Executing @Before advice on method");		
	}
	
	@Before("forDaoPackageNoGetterSetter()")
	public void performApiAnalytics() {
		System.out.println("\n=====>>> Performing API analytics");		
	}

```


# Aspect order

如何控制aspect执行的顺序？

![picture 26](/images/ae28948b768c7202c50ee7a7ffff64cbffb9703a3680f0f0c3bc3e5814fd49bc.png)

aspect1.java

``` java
@Aspect
@Component
@Order(1)
public class MyCloudLogAsyncAspect {

	@Before("com.luv2code.aopdemo.aspect.LuvAopExpressions.forDaoPackageNoGetterSetter()")
	public void logToCloudAsync() {
		System.out.println("\n=====>>> Logging to Cloud in async fashion");		
	}

}

```

aspect2.java

``` java
@Aspect
@Component
@Order(2)
public class MyDemoLoggingAspect {
	
	@Before("com.luv2code.aopdemo.aspect.LuvAopExpressions.forDaoPackageNoGetterSetter()")
	public void beforeAddAccountAdvice() {		
		System.out.println("\n=====>>> Executing @Before advice on method");		
	}
	
}
```


# Aspect spy on method argument

``` java
@Aspect
@Component
@Order(2)
public class MyDemoLoggingAspect {
	
	@Before("com.luv2code.aopdemo.aspect.LuvAopExpressions.forDaoPackageNoGetterSetter()")
	public void beforeAddAccountAdvice(JoinPoint theJoinPoint) {
		
		System.out.println("\n=====>>> Executing @Before advice on method");	
		
		// display the method signature
		MethodSignature methodSig = (MethodSignature) theJoinPoint.getSignature();
		
		System.out.println("Method: " + methodSig);
		
		// display method arguments
		
		// get args
		Object[] args = theJoinPoint.getArgs();
		
		// loop thru args
		for (Object tempArg : args) {
			System.out.println(tempArg);
			
			if (tempArg instanceof Account) {
				
				// downcast and print Account specific stuff
				Account theAccount = (Account) tempArg;
				
				System.out.println("account name: " + theAccount.getName());
				System.out.println("account level: " + theAccount.getLevel());								

			}
		}
		
	}
	
}
```

使用这种方法，我们就可以获取被aspect监控的function的信息（signiture，argument）
# AfterReturning

注释方式:

![picture 27](/images/0569b714ea862018face4cdb4bc8fb0b4cf3bfddbb3ad41c2c061dbebb34eeea.png)  

*注意*：returning中的变量应该要与funciton中的一样

``` java
// add a new advice for @AfterReturning on the findAccounts method
	
@AfterReturning(
		pointcut="execution(* com.luv2code.aopdemo.dao.AccountDAO.findAccounts(..))",
		returning="result")
public void afterReturningFindAccountsAdvice(
				JoinPoint theJoinPoint, List<Account> result) {
	
	// print out which method we are advising on 
	String method = theJoinPoint.getSignature().toShortString();
	System.out.println("\n=====>>> Executing @AfterReturning on method: " + method);
			
	// print out the results of the method call
	System.out.println("\n=====>>> result is: " + result);
	
}
```

在使用afterReturning修改function return value的时候，需要做好logging，不然会让使用者感到疑惑。

# AfterThrowing

注释方法

![picture 29](/images/46b4192e6af0cba1d99a7a95e485b555ff485400471e87fcebc049abd465cf6d.png)

虽然我们在AfterThrowing中读到了exception，但是我们并没组织exeception的传播。exception还是会向上抛出，如下图。

![picture 30](/images/eefc6dc47955574ed07b2a80fd524b3a1054f9666e18a24bf8745e9699012934.png)

# After
无论在执行成功还是失败的时候，都会执行。但是无法读取throwing error或者return data。只能读取joinpoint的数据。

# Around

*proxy过程*:

![picture 31](/images/b3ddf92ade5114709c52b50523fce60adcb9a32c2de5c033d9d9442d867185d9.png)  


*例子*：

这个例子里，我们使用@around advice记录了function运行的时间
``` java
@Around("execution(* com.luv2code.aopdemo.service.*.getFortune(..))")	
public Object aroundGetFortune(
		ProceedingJoinPoint theProceedingJoinPoint) throws Throwable {
	
	// print out method we are advising on
	String method = theProceedingJoinPoint.getSignature().toShortString();
	System.out.println("\n=====>>> Executing @Around on method: " + method);
	
	// get begin timestamp
	long begin = System.currentTimeMillis();
	
	// now, let's execute the method
	Object result = theProceedingJoinPoint.proceed();
	
	// get end timestamp
	long end = System.currentTimeMillis();
	
	// compute duration and display it
	long duration = end - begin;
	System.out.println("\n=====> Duration: " + duration / 1000.0 + " seconds");
	
	return result;
}
```

Around会给我们一个ProceedingJoinPoint。这个是执行的function的一个handler，我们可以用这个handler获取function的时间，并执行function。

## 使用@around advice来handle exception

``` java
	private Logger myLogger = Logger.getLogger(getClass().getName());
	
	@Around("execution(* com.luv2code.aopdemo.service.*.getFortune(..))")	
	public Object aroundGetFortune(
			ProceedingJoinPoint theProceedingJoinPoint) throws Throwable {
		
		// print out method we are advising on
		String method = theProceedingJoinPoint.getSignature().toShortString();
		myLogger.info("\n=====>>> Executing @Around on method: " + method);
		
		// get begin timestamp
		long begin = System.currentTimeMillis();
		
		// now, let's execute the method
		Object result = null;
		
		try {
			result = theProceedingJoinPoint.proceed();
		} catch (Exception e) {
			// log the exception
			myLogger.warning(e.getMessage());
			
			// give users a custom messagee
			result = "Major accident! But no worries, "
					+ "your private AOP helicopter is on the way!";
		}
		
		// get end timestamp
		long end = System.currentTimeMillis();
		
		// compute duration and display it
		long duration = end - begin;
		myLogger.info("\n=====> Duration: " + duration / 1000.0 + " seconds");
		
		return result;
	}
```

由于在around advice中我们可以直接call 目标function，所以我们可以handle exception。甚至可以选择直接吧exception给吞了。这样不是best practice。

