# Spring Basic

+ Inversion of Control (IoC)

The object initialzie is control by outside (Spring).
And at runtime base on configuration will inject object (bean) for the variable.

+ Dependency Injection (DI)

Different point of view for Inversion of Control,
Inejct object required dependency by Spring.
  
+ Bean Factory

IoC: Bean Factory is an object factory to create Object (Bean) and manage it.
DI: Then inject Bean to

ex (from www.luv2code.com):

**MyApp** can consolut different Coach *BaseballCoach*, *TrackCoach*, .. and so on to get **getDailyWorkout** suggestion

The problem here is even we still need manually control creation of Coach.

```java
public interface Coach {
    String getDailyWorkout();
}

class BaseballCoach implements Coach {
    @Override
    String getDailyWorkout() {
        return "Batting 300 times"
    }
}

class TrachCoach implements Coach {
    @Override
    String getDailyWorkout() {
        return "Keep Run";
    }
}

class MyApp {
    public static void main(String[] args) {
        Coach theCoach = new BaseballCoach();

        System.out.println(theCoach.getDailyWorkout());
    }
}
```

## Spring Container (Bean Factory++)

Spring Container

+ Bean Factory (Create and Manage Object and so on)
+ Inject Object
+ Web related (i18n)

### Configuration Spring Container (ApplicationContext)

+ XML configuration (legacy)
+ Java Annotations
+ Java Source Code

File: applicationContext.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- Define your beans here -->
    <bean id="myCoach"
          class="note.spring.springdemo.TrackCoach" >
    </bean>
</beans>
```

### Spring IoC and DI

1. Configure bean (XML, Annotations, Java Source)
2. Create a Spring Container
3. Retrive Beans from Container

```java
package note.spring.springdemo;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class HelloSpringApp {

    public static void main(String[] args) {

        // create spring container from configuration
        ClassPathXmlApplicationContext context =
            new ClassPathXmlApplicationContext("applicationContext.xml");

        // retrieve bean from spring container
        Coach tehCoach = context.getBean("myCoach", Coach.class);

        // call methods on bean
        System.out.println(tehCoach.getDailyWorkout());

        // close the context
        context.close();
    }
}
```