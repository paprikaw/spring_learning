# SpringMVC

## 配置课程所需jar包和xml文件

使用 [这里](../starterCodes/solution-code-spring-mvc-config-files/spring-mvc/starter-files/spring-mvc-demo/)的文件来初始化你的MVC demo

当然，spring的jar files也需要在lib里面

## XML文件的配置

mvc配置需要两个XML文件


Spring MVC配置
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://xmlns.jcp.org/xml/ns/javaee"
	xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
	id="WebApp_ID" version="3.1">

	<display-name>spring-mvc-demo</display-name>

	<absolute-ordering />

	<!-- Spring MVC Configs -->

	<!-- Step 1: Configure Spring MVC Dispatcher Servlet -->
	<servlet>
		<servlet-name>dispatcher</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/spring-mvc-demo-servlet.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>

	<!-- Step 2: Set up URL mapping for Spring MVC Dispatcher Servlet -->
	<servlet-mapping>
		<servlet-name>dispatcher</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
	
</web-app>
```

Spring 配置：
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans
    	http://www.springframework.org/schema/beans/spring-beans.xsd
    	http://www.springframework.org/schema/context
    	http://www.springframework.org/schema/context/spring-context.xsd
    	http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

	<!-- Step 3: Add support for component scanning -->
	<context:component-scan base-package="com.luv2code.springdemo" />

	<!-- Step 4: Add support for conversion, formatting and validation support -->
	<mvc:annotation-driven/>

	<!-- Step 5: Define Spring MVC view resolver -->
	<bean
		class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/WEB-INF/view/" />
		<property name="suffix" value=".jsp" />
	</bean>

</beans>

```

## MVC应用的开发步骤

![picture 1](/images/4cce9f51f09061ffb80641ca73c3060e0716ce6f6b559be2130ee6e8812706fa.png)  

## 创建controller class

``` java
// Step1: 创建controller class，并在上面标注@Controller
@Controller
public class HomeController {
	
    // Step 2. 在controller class中定义controller function，用来mapping到不同的path上
	@RequestMapping("/")
	public String showString () {
		return "main-menu";
	}
}
```

return的string是view的jsp文件名（不包括prefix和后缀）


## Modal

## Request param

``` java
@RequestMapping("/processFormVersionThree")	
	public String processFormVersionThree(
			@RequestParam("studentName") String theName, 
			Model model) {
				
		// convert the data to all caps
		theName = theName.toUpperCase();
		
		// create the message
		String result = "Hey My Friend from v3! " + theName;
		
		// add message to the model
		model.addAttribute("message", result);
				
		return "helloworld";
	}
```
在这个controller中，requesetParam直接将Request中的param赋值给theName变量，就可以直接使用了

传统的过程是：

``` java
	@RequestMapping("/processFormVersionThree")	
	public String processFormVersionThree(
			@RequestParam("studentName") String theName, 
			Model model) {
				
		// convert the data to all caps
		theName = theName.toUpperCase();
		
		// create the message
		String result = "Hey My Friend from v3! " + theName;
		
		// add message to the model
		model.addAttribute("message", result);
				
		return "helloworld";
	}	
```

## Controller level request mapping

可以在controllershang标注@RequestMapping，这样所有controller内部的mapping就都是相对于controller的路径了