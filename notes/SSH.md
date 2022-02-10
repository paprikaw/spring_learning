# 开发一个SSH应用

这个教程将跳过Spring MVC中的View部分，淡化hibernate部分，主要着眼于项目的部署和DAO的应用


## 配置

### Dependencies

* Spring
* Hibernate
* Spring MVC
* mysql connector

### 项目初始化
使用 eclipes dynamic web service 初始化，目录结构为

```
.
├── WebContent
│   ├── META-INF
│   │   └── MANIFEST.MF
│   └── WEB-INF
│       ├── lib
│       │   └── 所有Dependencies
│       ├── spring-mvc-crud-demo-servlet.xml
│       ├── view
│       │   └── list-customers.jsp
│       └── web.xml
├── build
│   └── ...
└── src
    └── com.luv2code.springdemo
    │   ├── controller
    │   ├── dao
    │   └── entity
    └── testdb
        └── TestDbServlet.java
```

其中lib目录中存放spring配置文件，具体文件在[这里](../class_solution_codes/04-spring-mvc-crud-5/web-customer-tracker-starter-files/WebContent/WEB-INF)

## 项目组成

* controller
* DAO, DAO implementation
* entity(hibernate)
* 前端jsp（忽略）
* Service layer

![picture 17](/images/0489209e589dc5430d6917454dde9fef6f5ace3bd3a54464ceb22789b247f10d.png)  



## DAO
编写DAO我们一般会先写一个DAO的interface，然后再写一个DAO的implementation。

### Interface:

``` java
package com.luv2code.springdemo.dao;

import java.util.List;

import com.luv2code.springdemo.entity.Customer;

public interface CustomerDAO {

	public List<Customer> getCustomers();
	
}
```

Implementation

``` java
package com.luv2code.springdemo.dao;

import java.util.List;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.query.Query;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.annotation.Transactional;

import com.luv2code.springdemo.entity.Customer;

@Repository
public class CustomerDAOImpl implements CustomerDAO {

	// need to inject the session factory
	@Autowired
	private SessionFactory sessionFactory;
			
	@Override
	@Transactional
	public List<Customer> getCustomers() {
		
		// get the current hibernate session
		Session currentSession = sessionFactory.getCurrentSession();
				
		// create a query
		Query<Customer> theQuery = 
				currentSession.createQuery("from Customer", Customer.class);
		
		// execute query and get result list
		List<Customer> customers = theQuery.getResultList();
				
		// return the results		
		return customers;
	}

}
```


### @Repository annotation

spring用来注释DAO的annotation。

### 将DAO注入到controller class

``` java
@Controller
@RequestMapping("/customer")
public class CustomerController {

	// need to inject the customer dao
	@Autowired
	private CustomerDAO customerDAO;
	
	@RequestMapping("/list")
	public String listCustomers(Model theModel) {
		
		// get customers from the dao
		List<Customer> theCustomers = customerDAO.getCustomers();
				
		// add the customers to the model
		theModel.addAttribute("customers", theCustomers);
		
		return "list-customers";
	}
	
}
```

## Service layer

我们在Controller与DAO层之间加一层service layer，来保证我们可以使用不同的DAO或者组合使用DAO


Service.java
``` java
public interface CustomerService {

	public List<Customer> getCustomers();
	
}
```

ServiceImpl.java
``` java
@Service
public class CustomerServiceImpl implements CustomerService {

	// need to inject customer dao
	@Autowired
	private CustomerDAO customerDAO;
	
	@Override
	@Transactional
    // 这里transactional注释在service层中，我们可以去掉customerDao中的transactional注释
	public List<Customer> getCustomers() {
		return customerDAO.getCustomers();
	}
} 
```

### *Transaction是加在DAO层中还是service层中？*
```
In general, if you want to run the the DAO methods in the same transaction then you can use @Transactional at the service layer. Here's a use case of using @Transactional at the service layer.

Say for example we have 

BankDAO

- deposit(...)

- withdraw(…)


BankServiceImpl

@Transactional

public void transferFunds(...) {

    deposit(...);

    withdraw(...);

}


If we are transferring funds, we want that to run in the same transaction. By making use of @Transactional at service layer, then we can have this transactional support and both methods will run in the same transaction. This would call deposit() and withdraw(). If either of those methods  failed then we'd want to roll the transaction back.

However, if we had @Transactional at DAO level instead of service level, then the methods deposit() and withdraw() would run in separate transactions. If one of them failed, then we would NOT be able to rollback the other method ... because it is in a separate transaction.This is not the desired approached.

Hence, to run DAO methods in the same transaction, make use of @Transactional at the service layer.  So that's one real-time project use case for applying @Transactional at the Service layer.

let me know if this clears your doubt.  :-) 
```

