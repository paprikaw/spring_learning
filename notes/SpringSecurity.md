# SpringSecurity 使用

## 1 Spring Security的流程:

![picture 38](/images/6a72cd437487498e49a8038022127c87ee0de1d8ad8e1e15b7d683398c9afead.png)  


## 2 Security 基本概念

![picture 39](/images/55540070a171325d527debb9b33a415fbcf0e3478d944c114da66fb81f734d35.png)  

## 3 Example overview

![picture 40](/images/8f2ce8d6013657f6d4d664ae6ce7e203e0568ab8e06707a270e2e632f92f90c6.png)  

我们有三个页面，分别开放给三个不同的权限人群


### *Project Steps*

![picture 41](/images/abe4ab574f677c1afc94d2be0a68e57b1b74f747512f137660480324ec7661a7.png)



### Prep works

1. 我们需要创建一个maven project，使用一个pom文件去定义需要什么dependencies。这个pom文件请参考[这里](../class_solution_codes/07-spring-security-5/solution-code-spring-security-demo-01-base-app/pom.xml)

2. 我们使用java code来配置文件，将config file放在 src/main/java/packagename/config 文件夹下:


    配置spring
    ``` java
    @Configuration
    @EnableWebMvc
    @ComponentScan(basePackages="com.luv2code.springsecurity.demo")
    public class DemoAppConfig {

    	// define a bean for ViewResolver

    	@Bean
    	public ViewResolver viewResolver() {
        
    		InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
    
    		viewResolver.setPrefix("/WEB-INF/view/");
    		viewResolver.setSuffix(".jsp");
    
    		return viewResolver;
    	}
    
    }

    ```

    配置Dispatcher servlet:

    ``` java
    package com.luv2code.springsecurity.demo.config;

    import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

    public class MySpringMvcDispatcherServletInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    	@Override
    	protected Class<?>[] getRootConfigClasses() {
    		// TODO Auto-generated method stub
    		return null;
    	}

    	@Override
    	protected Class<?>[] getServletConfigClasses() {
    		return new Class[] { DemoAppConfig.class };
    	}

    	@Override
    	protected String[] getServletMappings() {
    		return new String[] { "/" };
    	}

    }
    ```




# Logout

我们可以为spring security加上logout的config。logout之后，sringsecurity会invalidate user的session并移除session cookies。

# CSRF支持

spring security默认开启对所有request的CSRF认证

白旭第八周周报

本周工作内容：

1. 学习后端知识
2. 开始制作sso登陆查看用户缓存界面

下周工作内容：

1. 完成查看用户缓存界面
2. 配合组内布置任务
3. 继续学习后端知识


# 角色权限的管理

## 增加权限

我们在spring security的config file中增加几种不同的权限：

``` java
@Configuration
@EnableWebSecurity
public class DemoSecurityConfig extends WebSecurityConfigurerAdapter {

	@Override
	protected void configure(AuthenticationManagerBuilder auth) throws Exception {

		// add our users for in memory authentication
		
		UserBuilder users = User.withDefaultPasswordEncoder();
		
		auth.inMemoryAuthentication()
			.withUser(users.username("john").password("test123").roles("EMPLOYEE"))
			.withUser(users.username("mary").password("test123").roles("EMPLOYEE", "MANAGER"))
			.withUser(users.username("susan").password("test123").roles("EMPLOYEE", "ADMIN"));
	}

	@Override
	protected void configure(HttpSecurity http) throws Exception {

		http.authorizeRequests()
				.anyRequest().authenticated()
			.and()
			.formLogin()
				.loginPage("/showMyLoginPage")
				.loginProcessingUrl("/authenticateTheUser")
				.permitAll()
			.and()
			.logout().permitAll();
		
	}
		
}
```
## 对于不同的role，限制访问的权限

1. 对于不同的权限，创建不同的页面（jsp，跳过）
2. 创建不同的controller，对应不同的权限
3. 更新 springsecurity config

	``` java 
	@Override
	protected void configure(HttpSecurity http) throws Exception {

		http.authorizeRequests()
			.antMatchers("/").hasRole("EMPLOYEE")
			.antMatchers("/leaders/**").hasRole("MANAGER")
			.antMatchers("/systems/**").hasRole("ADMIN")
			.and()
			.formLogin()
				.loginPage("/showMyLoginPage")
				.loginProcessingUrl("/authenticateTheUser")
				.permitAll()
			.and()
			.logout().permitAll();

	}
	```
4. 对于不同的权限，显示不同的内容

# Spring security 与database结合

目前，我们只是简单的将用户权限写到内存中，现在我们希望使用database存储角色权限。

