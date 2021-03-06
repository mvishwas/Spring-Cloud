Spring cloud bus is used to update configuration changes to applications dynamically without bringing them down.

We know that clients can get the configurations using spring-cloud-config only at the startup time.

One solution could be, client applications could periodically poll the config server for changes, after all they send eureka heartbeats.

The best solution could be to push the changes from server to client, because config changes are rare and no need to waste resources.

This is exactly what spring cloud bus does. It pushes configuration changes to client applications via messaging technology, like AMQP.

Note: It means all client applications become subscribers to AMQP.

Spring Cloud Bus setup
----------------------
Here are the steps required to setup.

1)Add the following dependency to spring cloud config server.

	<dependency>
		<groupid>org.springframework.cloud</groupid>
		<artifactid>spring-cloud-starter-bus-amqp</artifactid>
	</dependency>

2)Add the same dependency to all client applications.

3)Install and run an amqp server, such as Rabbit MQ. 

Note: Spring cloud bus works automatically with Rabbit MQ on the localhost.

How it works
------------
1)To see the configuration changes first lets make some change in git repository.<br>
2)Make a POST request to /bus/refresh (because config server does not poll for changes)<br>
3)The /bus/refresh end point makes sure it reads the updated values.<br>
4)Broker will ensure the message delivery to all clients.<br>
5)Clients receive messages and refresh themselves.

How refresh works
-----------------
We know spring boot applications can refreshed at runtime by adding actuator dependency. Actuator provides a /refresh POST end point. This end point affects only the following.<br>
1)Beans marked with @ConfigurationProperties<br>
2)Beans marked with @RefreshScope<br>
3)Logging level

@ConfigurationProperties
------------------------
@ConfigurationProperties is introduced in Spring Boot. It is an easy alternative to multiple @Value annotations. Instead of having many @Value annotations at the properties, we can use @ConfigurationProperties annotation at the class level like below.

	@RestController
	@ConfigurationProperties(prefix="wordConfig")
	public class WordController{
		String article;
		String name;
		
		@RequestMapping("/article")
		public string getWord(){
			return name+ " : " + article;
		}
	}

Note: All properties in the properties file should prefix with the word as mentioned in the prefix attribute.

Note: spring boot is not very strict when binding properties. It works with camelcase or all capitals or multiple words seperated by underscore also fine. For ex, for firstName property, the value in properties file can be any of these. FIRSTNAME or FIRST_NAME or first_Name.

@RefreshScope
-------------
@RefreshScope is introduced in spring cloud. It just don't rebind the properties, it reloads the entire bean. It creates a new bean and safely discards the older one.

Note: The bean is not reloaded until it is first time invoked. That means bean reloading happens lazily.

How @RefreshScope works
-----------------------
Spring creates a proxy for the actual bean. proxy is dependency injected into other beans. on refresh, the target bean is changed to the newly created bean by dereferencing the older bean.

Note: The users of original bean can safely finish their work.

