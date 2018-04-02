
# Spring 4 MVC REST Controller  
from [this page](http://viralpatel.net/blogs/spring-4-mvc-rest-example-json/)
###### 1. Create a new Maven Project

```mvn archetype:create -DgroupId=net.viralpatel.spring -DartifactId=SpringRest -DarchetypeArtifactId=maven-archetype-webapp```

run following command and convert the project in Eclipse project.

```mvn eclipse:eclipse```

import the project in Eclipse.

###### 2. **Add Spring 4 MVC Maven dependencies (Update pom.xml)**

add first the maven dependencies for Spring 4 MVC REST in our pom.xml file.
```
	<properties>
	    <java-version>1.7</java-version>
	    <springframework.version>4.3.1.RELEASE</springframework.version>
	    <jackson.version>2.7.5</jackson.version>
	</properties>
	<dependency>
	    <groupId>org.springframework</groupId>
	    <artifactId>spring-webmvc</artifactId>
	    <version>${springframework.version}</version>
	</dependency>
	<dependency>
	    <groupId>com.fasterxml.jackson.core</groupId>
	    <artifactId>jackson-databind</artifactId>
	    <version>${jackson.version}</version>
	</dependency>
	<dependency>
	    <groupId>javax.servlet</groupId>
	    <artifactId>javax.servlet-api</artifactId>
	    <version>3.0.1</version>
	    <scope>provided</scope>
	</dependency>
```

###### 3. ** Set Annotation based Configuration for Spring 4 MVC REST **

These will bootstrap the spring mvc application and set package to scan controllers and resources
Create RestMVCConfig.java 
@EnableWebMvc, @ComponentScan and @Configuration annotations. 

```
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;

@Configuration
@EnableWebMvc
@ComponentScan(basePackages = "net.viralpatel.spring")
public class RestMVCConfig {
}
```

###### 4. Set Servlet 3 Java Configuration
Create AppInitializer class under config package. This class will replace web.xml and 
it will map the spring’s dispatcher servlet and bootstrap it.

/src/main/java/net/viralpatel/spring/config/AppInitializer.java

```java
import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

public class AppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

	@Override
	protected Class[] getRootConfigClasses() {
		return new Class[] { AppConfig.class };
	}

	@Override
	protected Class[] getServletConfigClasses() {
		return null;
	}

	@Override
	protected String[] getServletMappings() {
		return new String[] { "/" };
	}
}
```

We have configured the dispatcher servlet using standard Java based configuration instead of the older web.xml. 
Thus web.xml is no longer required and we can simply delete it.

###### 5. Creation du modèle
```java
public class Customer {
	private Long id;
	private String firstName;
	private String lastName;
	private String email;
	private String mobile;
	private Date dateOfBirth;

	public Customer(long id, String firstName, String lastName, String email, String mobile) {
		this.id = id;
		this.firstName = firstName;
		this.lastName = lastName;
		this.email = email;
		this.mobile = mobile;
		this.dateOfBirth = new Date();
	}

	public Customer() {
	}

	//..Getter and setter methods
}
```

###### 6. Create the Dummy Customer Data Access Object (DAO)
Instead of storing the customer data in database and to make this example 
simple, we will create a dummy data access object that will store customer 
details in a list. This DAO class can be easily replaced with Spring Data 
DAO or custom DAO. But for this example we will keep it easy.

The CustomerDAO contains methods list(), get(), create(), update() and delete() 
to perform CRUD operation on customers.

```java
import java.util.ArrayList;
import java.util.List;
import org.springframework.stereotype.Component;
import net.viralpatel.spring.model.Customer;

@Component
public class CustomerDAO {
	// Dummy database. Initialize with some dummy values.
	private static List customers;
	{
		customers = new ArrayList();
		customers.add(new Customer(101, "John", "Doe", "djohn@gmail.com", "121-232-3435"));
		customers.add(new Customer(201, "Russ", "Smith", "sruss@gmail.com", "343-545-2345"));
		customers.add(new Customer(301, "Kate", "Williams", "kwilliams@gmail.com", "876-237-2987"));
	}

	public List list() {
		return customers;
	}

	public Customer get(Long id) {
		for (Customer c : customers) {
			if (c.getId().equals(id)) {
				return c;
			}
		}
		return null;
	}

	public Customer create(Customer customer) {
		customer.setId(System.currentTimeMillis());
		customers.add(customer);
		return customer;
	}

	public Long delete(Long id) {
		for (Customer c : customers) {
			if (c.getId().equals(id)) {
				customers.remove(c);
				return id;
			}
		}
	return null;
	}

	public Customer update(Long id, Customer customer) {
		for (Customer c : customers) {
			if (c.getId().equals(id)) {
				customer.setId(c.getId());
				customers.remove(c);
				customers.add(customer);
				return customer;
			}
		}

		return null;
	}

}
```

###### 7. Create the Customer REST Controller
Now let us create CustomerRestController class. This class is annotated with 
@RestController annotation. Also note that we are using new annotations 
@GetMapping, @PostMapping, @PutMapping and @DeleteMapping instead of 
standard @RequestMapping. These annotations are available since 
Spring MVC 4.3 and are standard way of defining REST endpoints. They act as 
wrapper to @RequestMapping. For example @GetMapping is a composed annotation 
that acts as a shortcut for @RequestMapping(method = RequestMethod.GET).

```java
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;
import net.viralpatel.spring.dao.CustomerDAO;
import net.viralpatel.spring.model.Customer;

@RestController
public class CustomerRestController {
	@Autowired
	private CustomerDAO customerDAO;

	@GetMapping("/customers")
	public List getCustomers() {
		return customerDAO.list();
	}

	@GetMapping("/customers/{id}")
	public ResponseEntity getCustomer(@PathVariable("id") Long id) {
		Customer customer = customerDAO.get(id);
		if (customer == null) {
			return new ResponseEntity("No Customer found for ID " + id, HttpStatus.NOT_FOUND);
		}
	return new ResponseEntity(customer, HttpStatus.OK);
	}

	@PostMapping(value = "/customers")
	public ResponseEntity createCustomer(@RequestBody Customer customer) {
		customerDAO.create(customer);
		return new ResponseEntity(customer, HttpStatus.OK);
	}

	@DeleteMapping("/customers/{id}")
	public ResponseEntity deleteCustomer(@PathVariable Long id) {
		if (null == customerDAO.delete(id)) {
			return new ResponseEntity("No Customer found for ID " + id, HttpStatus.NOT_FOUND);
		}
	return new ResponseEntity(id, HttpStatus.OK);
	}

	@PutMapping("/customers/{id}") 
	public ResponseEntity updateCustomer(@PathVariable Long id, @RequestBody Customer customer) {
		customer = customerDAO.update(id, customer);
		if (null == customer) {
			return new ResponseEntity("No Customer found for ID " + id, HttpStatus.NOT_FOUND);
		}
	return new ResponseEntity(customer, HttpStatus.OK);
	}

}
```








Spring @RequestHeader Annotation example
BY VIRAL PATEL · DECEMBER 27, 2013

Let us quickly check how to access http Header information in Spring MVC Controller.

Spring @RequestHeader Annotation
Spring MVC provides annotation @RequestHeader that can be used to map controller parameter to request header value. Following is the simple usage of spring @RequestHeader annotation.


 
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestHeader;
import org.springframework.web.bind.annotation.RequestMapping;
//..

@Controller
public class HelloController {

	@RequestMapping(value = "/hello.htm")
	public String hello(@RequestHeader(value="User-Agent") String userAgent)

		//..
	}
}
In above code snippet we defined a controller method hello() which is mapped to URL /hello.htm. Also we bind the parameter String userAgent using @RequestHeader annotation. When spring maps the request, it checks http header with name “User-Agent” and bind its value to String userAgent.

If the header value that you specified does not exists in request, Spring will initialize the parameter with null value. In case you want to set default value of parameter you can do so using defaultParameter attribute of spring @RequestHeader annotation.

@RequestMapping(value = "/hello.htm")
public String hello(@RequestHeader(value="User-Agent", defaultValue="foo") String userAgent)

	//..
}
Complete Tutorial
Now we know the concept, let us use it and create a Spring MVC application to print value of different http request headers. Following is demo application for Spring Requestheader example.

For this tutorial I will be using following tools and technologies:

Spring MVC 3.2.6.RELEASE
Java 6
Eclipse
Maven 3
Following is the project structure.
spring @requestheader project structure

Create and copy following file contents in the project structure.

Maven configuration: pom.xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>net.viralpatel.spring</groupId>
	<artifactId>Spring_Cookie_example</artifactId>
	<packaging>war</packaging>
	<version>1.0.0-SNAPSHOT</version>
	<name>Spring_RequestHeader</name>
	<properties>
		<org.springframework.version>3.2.6.RELEASE</org.springframework.version>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<compileSource>1.6</compileSource>
	</properties>
	<dependencies>
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>servlet-api</artifactId>
			<version>2.5</version>
			<scope>provided</scope>
		</dependency>
		<!-- Spring MVC  -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-web</artifactId>
			<version>${org.springframework.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>${org.springframework.version}</version>
		</dependency>
		<!-- JSTL taglib -->
		<dependency>
			<groupId>taglibs</groupId>
			<artifactId>standard</artifactId>
			<version>1.1.2</version>
		</dependency>
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>jstl</artifactId>
			<version>1.1.2</version>
		</dependency>
	</dependencies>
	<build>
		<finalName>springcookie</finalName>
	</build>
	<profiles>
	</profiles>
</project>
Maven configuration is simple. We just need Spring MVC and JSTL dependency.

Deployment description: web.xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	id="WebApp_ID" version="2.5">
	<display-name>Spring MVC Http Cookie</display-name>
	<welcome-file-list>
		<welcome-file>hello.htm</welcome-file>
	</welcome-file-list>

	<servlet>
		<servlet-name>spring</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<load-on-startup>1</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>spring</servlet-name>
		<url-pattern>*.htm</url-pattern>
	</servlet-mapping>

</web-app>
Web.xml is quite simple too. We just need to configure Spring’s DispatcherServlet with *.htm url pattern.

Spring Configuration: spring-servlet.xml
<?xml  version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd 
		http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">

	<context:annotation-config />
	<context:component-scan base-package="net.viralpatel.spring" />

	
</beans>
In spring-servlet.xml we just defined component scan to load @Controller classes. Note that we are not defining view resolver as we don’t need that for this example. But ideally you’ll have one view resolver that maps to JSP views.

Spring Controller: HelloController.java
package net.viralpatel.spring;

import javax.servlet.http.HttpServletResponse;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestHeader;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class HelloController {

	@RequestMapping(value = "/hello.htm")
	public String hello(
			@RequestHeader(value="Accept") String accept,
			@RequestHeader(value="Accept-Language") String acceptLanguage,
			@RequestHeader(value="User-Agent", defaultValue="foo") String userAgent,
			HttpServletResponse response) {

		System.out.println("accept: " + accept);
		System.out.println("acceptLanguage: " + acceptLanguage);
		System.out.println("userAgent: " + userAgent);
		
		return null;
	}

}
The logic to read http requestheader is written in HelloController. We used spring @RequestHeader annotation to map userAgent, accept, acceptLanguage strings from request header.
