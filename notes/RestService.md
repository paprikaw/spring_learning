# Rest Service

为什么需要Rest service？应用和应用之间可以用一种共同的语言交流，有利于前后端的分离

# Java Json data binding with jackson
    将json转换为java pojo，或者反其道而行之

我们可以认为的调用jackson api来读取json然后转化为pojo。在spring中，spring整合了jackson的能力，可以自动为我们转化json到pojo。

具体请参看jackson api。General speaking，如果我们要认为的调用jackson，我们需要首先定义存放数据的pojo，然后将json从某一个stream内读入，使用jackson转化为object，然后再使用这个object。

# spring的@RestController会在json和pojo之间转换

一个 Spring rest service其实并没有什么不同，只是controller返回的东西从render好的页面变成了rest api


# Exception handling
使用rest api的时候，我们经常会遇到各种错误（从数据库retrieve data错误，客户端发来的值效验错误。我们需要一种途径向客户端传送可靠的error数据。通常，exception handling有以下几个步骤：

1. 创建自定义的error entity，这个error entity会被exception handler拿来转换成json发给客户端。

    StudentErrorResponse.java
    ``` java
    public class StudentErrorResponse {

    	private int status;
    	private String message;
    	private long timeStamp;

    	public StudentErrorResponse() {
        
    	}
    
    	public StudentErrorResponse(int status, String message, long timeStamp) {
    		this.status = status;
    		this.message = message;
    		this.timeStamp = timeStamp;
    	}

    	public int getStatus() {
    		return status;
    	}

    	public void setStatus(int status) {
    		this.status = status;
    	}

    	public String getMessage() {
    		return message;
    	}

    	public void setMessage(String message) {
    		this.message = message;
    	}

    	public long getTimeStamp() {
    		return timeStamp;
    	}

    	public void setTimeStamp(long timeStamp) {
    		this.timeStamp = timeStamp;
    	}
    
    
    }
    ```
2. 创建custom的exception用来表示当前的错误

    StudentNotFoundException.java
    ``` java
    package com.luv2code.springdemo.rest;

    public class StudentNotFoundException extends RuntimeException {

    	public StudentNotFoundException(String message, Throwable cause) {
    		super(message, cause);
    	}

    	public StudentNotFoundException(String message) {
    		super(message);
    	}

    	public StudentNotFoundException(Throwable cause) {
    		super(cause);
    	}
    
    }
    ```

3. 定义exceptionHandler

    这个可以定义在controller class内部，exceiptionhandler可以为特定的exception也可以为generic的。

    ``` java
    @ExceptionHandler
    public ResponseEntity<StudentErrorResponse> handleException(StudentNotFoundException exc) {
    
    	StudentErrorResponse error = new StudentErrorResponse();
    
    	error.setStatus(HttpStatus.NOT_FOUND.value());
    	error.setMessage(exc.getMessage());
    	error.setTimeStamp(System.currentTimeMillis());
    
    	return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }
    ```

4. 在controller中根据具体情况throw exception
   
    ``` java
    @GetMapping("/students/{studentId}")
	public Student getStudent(@PathVariable int studentId) {
		
		// just index into the list ... keep it simple for now
		
		// check the studentId against list size
		
		if ( (studentId >= theStudents.size()) || (studentId < 0) ) {			
			throw new StudentNotFoundException("Student id not found - " + studentId);
		}
		
		return theStudents.get(studentId);
	}
 
    ```


## 使用AOP的方式来添加全局exception handling

现在的这种配置方式有许多的问题。第一，exception handler散落在各个controller中，管理起来比较困难。第二，创建的exception handler无法复用。这似乎是一个AOP的一个完美的使用用例。

我们将exceptionHandler抽取出来到一个新的class里面:

``` java
@ControllerAdvice
public class StudentRestExceptionHandler {

	// add exception handling code here
	
	// Add an exception handler using @ExceptionHandler
	
	@ExceptionHandler
	public ResponseEntity<StudentErrorResponse> handleException(StudentNotFoundException exc) {
		
		// create a StudentErrorResponse
		
		StudentErrorResponse error = new StudentErrorResponse();
		
		error.setStatus(HttpStatus.NOT_FOUND.value());
		error.setMessage(exc.getMessage());
		error.setTimeStamp(System.currentTimeMillis());
		
		// return ResponseEntity
		
		return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
	}
	
	// add another exception handler ... to catch any exception (catch all)
	
	@ExceptionHandler
	public ResponseEntity<StudentErrorResponse> handleException(Exception exc) {
		
		// create a StudentErrorResponse
		StudentErrorResponse error = new StudentErrorResponse();
		
		error.setStatus(HttpStatus.BAD_REQUEST.value());
		error.setMessage(exc.getMessage());
		error.setTimeStamp(System.currentTimeMillis());
		
		// return ResponseEntity		
		return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
	}
	
}
```
注意，我们使用@ControllerAdvice 来注释这个class。

只需要这样，所有的controller就都能使用这些定义好的exception handler了。


# Api Design

这个我们可以参考刘老师关于API的文档整理。总的来说，这里关于Rest API的讨论在于我们使用http action中的方法，结合url来设计api