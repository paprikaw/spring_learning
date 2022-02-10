# Hibernate Basics

## 配置环境

1. 下载Hibernate的jar包

    hibernate.org

2. 将hibernate的jar包放到项目的lib里面


*注意*：如果使用java 9以上的版本，Hibernate遇到问题可以去[这里](../FAQ/Section%2020%20-%20Hibernate%20Configuration%20with%20Annotations/182-heads-up-for-java-9-users.pdf)
## 配置流程

![picture 2](/images/49a271424f2b96834226e58cb1c4b2f7a6b1cd48cfc91570a069206d6ad78c0e.png)  

## 配置文件
``` xml
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>

    <session-factory>

        <!-- JDBC Database connection settings -->
        <property name="connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="connection.url">jdbc:mysql://localhost:3306/hb_student_tracker?useSSL=false&amp;serverTimezone=UTC</property>
        <property name="connection.username">hbstudent</property>
        <property name="connection.password">hbstudent</property>

        <!-- JDBC connection pool settings ... using built-in test pool -->
        <property name="connection.pool_size">1</property>

        <!-- Select our SQL dialect 每一个sql语言都与标准不太一样，所以需要调整使用的SQL server -->
        <property name="dialect">org.hibernate.dialect.MySQLDialect</property>

        <!-- Echo the SQL to stdout -->
        <property name="show_sql">true</property>

		<!-- Set the current session context -->
		<property name="current_session_context_class">thread</property>
 
    </session-factory>

</hibernate-configuration>
```

## Hibernate Mapping 配置

### *Terminology*
*Entity class*: 使用Hibernate map到database的class

### *XML configuration*
Legacy, 现在基本不用了

### *Annotaton Configuration*

*步骤*
``` java
package com.luv2code.hibernate.demo.entity;

// (*1)
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;

// 1. 将class map到database table中
@Entity
@Table(name="student")
public class Student {

// 2. 将class中的field map到table中
	@Id //注释表明primary key
	@GeneratedValue(strategy=GenerationType.IDENTITY) 
	@Column(name="id") // colunm的名字
	private int id;
	
	@Column(name="first_name")
	private String firstName;
	
	@Column(name="last_name")
	private String lastName;
	
	@Column(name="email")
	private String email;
	
	public Student() {
		
	}

	public Student(String firstName, String lastName, String email) {
		this.firstName = firstName;
		this.lastName = lastName;
		this.email = email;
	}

	public int getId() {
		return id;
	}

	public void setId(int id) {
		this.id = id;
	}

	public String getFirstName() {
		return firstName;
	}

	public void setFirstName(String firstName) {
		this.firstName = firstName;
	}

	public String getLastName() {
		return lastName;
	}

	public void setLastName(String lastName) {
		this.lastName = lastName;
	}

	public String getEmail() {
		return email;
	}

	public void setEmail(String email) {
		this.email = email;
	}

	@Override
	public String toString() {
		return "Student [id=" + id + ", firstName=" + firstName + ", lastName=" + lastName + ", email=" + email + "]";
	}
}
```
*注意*:

colunm name和primative值的名字不一定需要一样。如果不填写就默认使用primitive的名字当作colunm的名字

*问题*：

* [如何使用java code配置hibernate？](https://www.dineshonjava.com/hibernate/hbm2ddl-configuration-and-name/)

*注释*：

 1. [为什么使用Hibernate的Annotation？](../FAQ/Section%2020%20-%20Hibernate%20Configuration%20with%20Annotations/185-faq-why-we-are-using-jpa-annotation-instead-of-hibernate.pdf)


## Session factory 和 Session
![picture 3](/images/3f4a083a134fad4310a2c645879db679758bd54b017f4b6170b11ae63375dd59.png)  

## 开发一个hibernate应用的步骤

1. 配置并定义好hibernate的entity (同上j)
2. 在main funciton里调用
    ``` java
    	public static void main(String[] args) {

    		// create session factory
    		SessionFactory factory = new Configuration()
    								.configure("hibernate.cfg.xml")
    								.addAnnotatedClass(Student.class)
    								.buildSessionFactory();
    		// create session
    		Session session = factory.getCurrentSession();
    
    		try {			
    			// create a student object
    			System.out.println("Creating new student object...");
    			Student tempStudent = new Student("Paul", "Doe", "paul@luv2code.com");
    
    			// start a transaction
    			session.beginTransaction();
    
    			// save the student object
    			System.out.println("Saving the student...");
    			session.save(tempStudent);
    
    			// commit transaction
    			session.getTransaction().commit();
    
    			System.out.println("Done!");
    		}
    		finally {
    			factory.close();
    		}
    	}
    ```

## Primary key and hibernate

![picture 4](/images/71d1517820633e469a576e88a35a92b96a06e05fd058258bfbeb0d9589b0c661.png)  

在SQL中，我们可以定义Primery key的generate strategy。在hibernate中，我们可以使用annotation做到这一点:

``` java
@Id
@GeneratedValue(strategy=GenerationType.IDENTITY)
@Column(name="id")
private int id;
```


## 从hibernate中retrieve object

``` java
// get a new session and start transaction
session = factory.getCurrentSession();
session.beginTransaction();
			
// >> retrieve student based on the id: primary key
Student myStudent = session.get(Student.class, tempStudent.getId());
			
// commit the transaction
session.getTransaction().commit();
```

## Query objects


``` java
// start a transaction
session.beginTransaction();
		
// query students
List<Student> theStudents = session.createQuery("from Student").getResultList();
```

更多的Hibernate SQL语句可以参考官网

### *问题*
[如何获得hibernate更详细的log？](../FAQ/Section%2021%20-%20Hibernate%20CRUD%20Features_%20Create,%20Read,%20Update%20and%20Delete/196-faq-how-to-view-hibernate-sql-parameter-values.pdf)

## Update objects

### *两种方式来update*
``` java
int studentId = 1;
			
// >>>>> 1. 使用直接使用hibernate entety的setter function

// now get a new session and start transaction
session = factory.getCurrentSession();
session.beginTransaction();
			
// retrieve student based on the id: primary key
Student myStudent = session.get(Student.class, studentId);
myStudent.setFirstName("Scooby");
			
// commit the transaction
session.getTransaction().commit();

// >>>>> 2. 使用 hibernate sql

session = factory.getCurrentSession();
session.beginTransaction();
			
// update email for all students
session.createQuery("update Student set email='foo@gmail.com'")
	.executeUpdate();
			
// commit the transaction
session.getTransaction().commit();
```

## Delete Object

``` java
int studentId = 1;
// now get a new session and start transaction
session = factory.getCurrentSession();
session.beginTransaction();

// >>>>> 使用delete函数删除	

// retrieve student based on the id: primary key
Student myStudent = session.get(Student.class, studentId);
			
// delete the student
session.delete(myStudent);

// >>>>> 使用hibernate sql删除

session.createQuery("delete from Student where id=2").executeUpdate();

// commit the transaction
session.getTransaction().commit();
```

# Advanced Mapping
## Foreign key

```
将两个表链接的key。我们可以利用foreign key创建one to one或者one to many等等的关系。
```
### *Foreign key的作用*

![picture 6](/images/b623b92e3908ac81a64733b0fef498355dba743e33b87566cf4a17d9d5969732.png)  


### *Foreign key例子*
![picture 5](/images/60ba8b0139366f3c99d47d54dd48cf92822f37cc5b5e945f4ae378413be5e546.png)  



## Entity lifecycle

![picture 8](/images/52f3f768bdaad1dd0a3dcdb991a46abbf0c421323374fb6999d5821def6e5a2c.png)  

![picture 9](/images/3d303deff55ad3da99535346b7b9a7eb0b614ff80facb534fd55a0ba29e7d6e5.png)  

## One to one mapping

### *使用one to one mapping的步骤*


1. 定义好一个database，包括primary key，foreign key等等。
具体请参考这个[文件夹里的scripts](../class_solution_codes/03-hibernate-5/hibernate-mapping-database-scripts)

2. 定义class object来映射到database中

    * *cascade type*
        ![picture 10](/images/c9e26ae14fc054dc2a0c1f9632f1e2634bc1706fcd575a09cc212250b10d6dba.png)  

    * 例子:
        ``` java
        package com.luv2code.hibernate.demo.entity;

        import javax.persistence.Column;
        import javax.persistence.Entity;
        import javax.persistence.GeneratedValue;
        import javax.persistence.GenerationType;
        import javax.persistence.Id;
        import javax.persistence.Table;

        // Step 1: annotate the class as an entity and map to db table
        @Entity
        @Table(name="instructor_detail")
        public class InstructorDetail {
        
        	// Step 2: define the fields
        	// annotate the fields with db column names
        	@Id
        	@GeneratedValue(strategy=GenerationType.IDENTITY)
        	@Column(name="id")
        	private int id;

        	@Column(name="youtube_channel")
        	private String youtubeChannel;

        	@Column(name="hobby")
        	private String hobby;

        	// create constructors
        	public InstructorDetail() {
            
        	}

        	public InstructorDetail(String youtubeChannel, String hobby) {
        		this.youtubeChannel = youtubeChannel;
        		this.hobby = hobby;
        	}

        	// generate getter/setter methods
        	public int getId() {
        		return id;
        	}

        	public void setId(int id) {
        		this.id = id;
        	}

        	public String getYoutubeChannel() {
        		return youtubeChannel;
        	}

        	public void setYoutubeChannel(String youtubeChannel) {
        		this.youtubeChannel = youtubeChannel;
        	}

        	public String getHobby() {
        		return hobby;
        	}

        	public void setHobby(String hobby) {
        		this.hobby = hobby;
        	}

        	// generate toString() method
        	@Override
        	public String toString() {
        		return "InstructorDetail [id=" + id + ", youtubeChannel=" + youtubeChannel + ", hobby=" + hobby + "]";
        	}

        }
        ```
3. 将不同的object 通过foreign key连接起来

    在父表中定义字表并注释cascade
    ``` java
    // 在定义foreign key的时候使用一下注释
    @OneToOne(cascade=CascadeType.ALL)
    @JoinColumn(name="instructor_detail_id")
    // The @JoinColumn annotation is used to specify the FOREIGN KEY column used when joining an entity association or an embeddable collection.
    private InstructorDetail instructorDetail;
    ```

    我们在database中已经定义好了两张表之间的关系:
    ``` sql
    CREATE TABLE `instructor_detail` (
        # Some definition
    ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=latin1;

    DROP TABLE IF EXISTS `instructor`;

    CREATE TABLE `instructor` (
        # Some other operation...
        CONSTRAINT `FK_DETAIL` FOREIGN KEY (`instructor_detail_id`) REFERENCES `instructor_detail` (`id`) ON DELETE NO ACTION ON UPDATE NO ACTION
    ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=latin1;
    ```

    所以我们在使用hibernate注释的时候，只需要定义cascade type和foreign key
## Bidirection relationship of one-to-one

在one-to-one中，单项的关系可以表现为 `instructor -> instructor_details`,这种关系在子表中无法retrieve父表。我们使用bidirectonal的关系`instructor <-> instructor_details`就可以实现子表中对父表的retrieve

步骤：
1. 在子表中申明父表并使用hibernate注释
    ``` java
        // mappedBy申明了在父表中，InstructorDetail这个object存在instructorDetail这个变量中，hibernate会使用这个注释找到拥有对应instructorDetails的instructor    
    	@OneToOne(mappedBy="instructorDetail", cascade=CascadeType.ALL)
    	private Instructor instructor;

    
    	public Instructor getInstructor() {
    		return instructor;
    	}

    	public void setInstructor(Instructor instructor) {
    		this.instructor = instructor;
    	}
    ```
    ### *关于mappedBy*
    ![picture 12](/images/c1dbdc8a48949f601073567daf3e11d2b2da71584a77471d9cd890fad57fc1e6.png)  

2. 在main application中使用这种bidirectonal关系

    现在，我们可以在获取的instructor_detail中retrieve到instructor了

### *Issues*

在删除bidirectional的表时如果不使用cascade删除的话，就不能直接删除。我们需要:
``` java
// remove the associated object reference
// break bi-directional link
tempInstructorDetail.getInstructor().setInstructorDetail(null);
			
session.delete(tempInstructorDetail);
```

如图所示：

![picture 11](/images/89fdfb3b51d2809c2bb9fc6de4566b090163e85d7ca55cb9a1a5c7622d92f87d.png)  

需要先删除instructor对象中对instructor detail的关系，因为instructor detail删除了，但是instructor并没有删除。


## One-to-many bidirectional


在这里，我们创建一个course table，它和instructor有如下的关系:

![picture 13](/images/096528f68d03287fed9a660c8140d138e597201689a9dd9e534ec706affc5f9f.png)  

这种关系我们称为one-to-many，而且是bidirectional的。在创建这种关系的时候，我们还有额外的要求：cascade type不能有delete。因为删除instructor不代表课程消失，反之亦然。


步骤

1. 创建database并创建一些table, 并且说明其中foreign key的一些关系

    这一步可以参考这个[sql script](../class_solution_codes/03-hibernate-5/hibernate-mapping-database-scripts/hb-03-one-to-many/create-db.sql)

2. 创建相关的class，并使用annotation

    Course.class:

    ``` java
    package com.luv2code.hibernate.demo.entity;
    import javax.persistence.CascadeType;
    import javax.persistence.Column;
    import javax.persistence.Entity;
    import javax.persistence.GeneratedValue;
    import javax.persistence.GenerationType;
    import javax.persistence.Id;
    import javax.persistence.JoinColumn;
    import javax.persistence.ManyToOne;
    import javax.persistence.Table;

    @Entity
    @Table(name="course")
    public class Course {
    	// Define some fields...

        // 定义Instructor，说明many-to-one的类型和cascade的类型
    	@ManyToOne(cascade= {CascadeType.PERSIST, CascadeType.MERGE,
    						 CascadeType.DETACH, CascadeType.REFRESH})
    	@JoinColumn(name="instructor_id")
    	private Instructor instructor;
    
    	public Instructor getInstructor() {
    		return instructor;
    	}

    	public void setInstructor(Instructor instructor) {
    		this.instructor = instructor;
    	}

    	@Override
    	public String toString() {
    		return "Course [id=" + id + ", title=" + title + "]";
    	}
    }
    ```

    Instructor.java:
    ``` java
    package com.luv2code.hibernate.demo.entity;
    import java.util.ArrayList;
    import java.util.List;

    import javax.persistence.CascadeType;
    import javax.persistence.Column;
    import javax.persistence.Entity;
    import javax.persistence.GeneratedValue;
    import javax.persistence.GenerationType;
    import javax.persistence.Id;
    import javax.persistence.JoinColumn;
    import javax.persistence.OneToMany;
    import javax.persistence.OneToOne;
    import javax.persistence.Table;

    @Entity
    @Table(name="instructor")
    public class Instructor {

    	// annotate the class as an entity and map to db table
    
    	// define the fields
    
    	// annotate the fields with db column names
    
    	// ** set up mapping to InstructorDetail entity
    
    	// create constructors
    
    	// generate getter/setter methods
    
    	// generate toString() method

        // 定义course，关系为one-to-many
    	@OneToMany(mappedBy="instructor",
    			   cascade= {CascadeType.PERSIST, CascadeType.MERGE,
    						 CascadeType.DETACH, CascadeType.REFRESH})
    	private List<Course> courses;
    	public List<Course> getCourses() {
    		return courses;
    	}

    	public void setCourses(List<Course> courses) {
    		this.courses = courses;
    	}
    
    	// add convenience methods for bi-directional relationship
    
    	public void add(Course tempCourse) {
        
    		if (courses == null) {
    			courses = new ArrayList<>();
    		}
    
    		courses.add(tempCourse);
    
    		tempCourse.setInstructor(this);
    	}
    }
    ```

3. 创建instructor，instructor_details, 使用setter function将details装到instructor里面。然后利用instructor的add function将courses也装到instructor里面。

注意：我们需要将course和instructor联系起来，他们之间的关系是bidirectional, one-to-many。这种情况下，在course中可以直接使用setInstructor来设置对应的instructor，但是在instructor中没有类似的setter。所以我们通常会在instructor中增加一个addCourse函数，实现双向的添加。

4. retrieve instructor和course

    ``` java
    package com.luv2code.hibernate.demo;

    import org.hibernate.Session;
    import org.hibernate.SessionFactory;
    import org.hibernate.cfg.Configuration;

    import com.luv2code.hibernate.demo.entity.Course;
    import com.luv2code.hibernate.demo.entity.Instructor;
    import com.luv2code.hibernate.demo.entity.InstructorDetail;

    public class GetInstructorCoursesDemo {

    	public static void main(String[] args) {

    		// create session factory
    		SessionFactory factory = new Configuration()
    								.configure("hibernate.cfg.xml")
    								.addAnnotatedClass(Instructor.class)
    								.addAnnotatedClass(InstructorDetail.class)
    								.addAnnotatedClass(Course.class)
    								.buildSessionFactory();
    
    		// create session
    		Session session = factory.getCurrentSession();
    
    		try {			
            
    			// start a transaction
    			session.beginTransaction();
    
    			// get the instructor from db
    			int theId = 1;
    			Instructor tempInstructor = session.get(Instructor.class, theId);		
    
    			System.out.println("Instructor: " + tempInstructor);
    
    			// get courses for the instructor
    			System.out.println("Courses: " + tempInstructor.getCourses());
    
    			// commit transaction
    			session.getTransaction().commit();
    
    			System.out.println("Done!");
    		}
    		finally {
            
    			// add clean up code
    			session.close();
    
    			factory.close();
    		}
    	}

    }

    ```

## Eager and lazy loading

不同关系之间的default loading mode

![picture 14](/images/1edb95a8751768761522854af1fdc0d26bd0470739d9b2591a22d4a79508c77b.png)

*注意*: lazy loading需要open session，如果session close，那么就会throw error。

以下是不同的lazy loading的方法:

![picture 15](/images/4cb16ea2383e5f629c1c9989b1e06eff8af9b4fc38c5e0d68610395f358f71b6.png)  


使用这种方法，我们可以在一个session中retrieve一个父表，然后在另外一个session里面retrieve夫表相关的子表。

具体配置：

1. 在配置entity时设置lazy loading的annotation:

    ``` java
    @OneToMany(fetch=FetchType.LAZY,
    		   mappedBy="instructor",
    		   cascade= {CascadeType.PERSIST, CascadeType.MERGE,
    					 CascadeType.DETACH, CascadeType.REFRESH})
    private List<Course> courses;
    ```

2. 在session open的时候进行lazy loading的操作，其中可以使用两种方法将数据加载到memory里面以便session closed之后数据还能被retrieve:
   1. 在session open时retrieve，之后再retrieve
        ``` java
        // start a transaction
        session.beginTransaction();

        // get the instructor from db
        int theId = 1;
        Instructor tempInstructor = session.get(Instructor.class, theId);		

        System.out.println("luv2code: Instructor: " + tempInstructor);

        System.out.println("luv2code: Courses: " + tempInstructor.getCourses());

        // commit transaction
        session.getTransaction().commit();

        // close the session
        session.close();

        System.out.println("\nluv2code: The session is now closed!\n");

        // option 1: call getter method while session is open

        // get courses for the instructor
        System.out.println("luv2code: Courses: " + tempInstructor.getCourses());

        System.out.println("luv2code: Done!");
        ```
        注意：这里在session open的时候就已经call了一次getCourses，之后close之后再call一次就不会报错
    2. 使用HQL直接将子表的数据join到夫表上
        ``` java
        // start a transaction
        session.beginTransaction();
    
        // option 2: Hibernate query with HQL
    
        // get the instructor from db
        int theId = 1;

        Query<Instructor> query = 
        		session.createQuery("select i from Instructor i "
        						+ "JOIN FETCH i.courses "
        						+ "where i.id=:theInstructorId", 
        				Instructor.class);

        // set parameter on query
        query.setParameter("theInstructorId", theId);
    
        // execute query and get instructor
        Instructor tempInstructor = query.getSingleResult();
    
        System.out.println("luv2code: Instructor: " + tempInstructor);	
    
        // commit transaction
        session.getTransaction().commit();
    
        // close the session
        session.close();
    
        System.out.println("\nluv2code: The session is now closed!\n");
    
        // get courses for the instructor
        System.out.println("luv2code: Courses: " + tempInstructor.getCourses());
        ```

### *Heads up*

什么是hibernate中的session？




## Uni-directional one-to-many relationship

现在，对于每个course，我们希望创建很多个reviews。在course中我们可以retrieve reviews，但是reviews本身不知道course是什么。这需要用到这种关系。

具体实现请参考[这里](../class_solution_codes/03-hibernate-5/solution-code-hibernate-hb-04-one-to-many-uni)


## Many-to-many relationship

![picture 16](/images/61327208dbee0a6bfd3b2303a3609bb94c46368b397f14c57569cc87f3cf9790.png)  