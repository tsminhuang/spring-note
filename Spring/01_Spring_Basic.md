# Spring Basic

+ Inversion of Control (IoC)

  The object initialzie is control by outside (Spring).
  And at runtime base on configuration will inject object (Bean) for the variable.

+ Dependency Injection (DI)

  Different point of view for Inversion of Control.
  Inejct required dependency by Spring.
  
+ Bean Factory
  + IoC: Bean Factory is an object factory to create Object (Bean) and manage it.
  + DI: IoC create B for us and inejct B into A.

ex (from www.luv2code.com):

**MyApp** can consolut different Coach *BaseballCoach*, *TrackCoach*, .. and so on to get **getDailyWorkout** suggestion.

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

## Configuration Spring Container (ApplicationContext)

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

### Spring Container (Bean Factory++)

Spring Container is very similiar Bean Factory but add some extra support for web like: i18n.

1. Configure Bean (XML, Annotations, Java Source)
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

### Dependency Injection

***Dependency***: A depends on B means A rely on B for provide service.

***Constructor Inejection***

1. Define dependency interface and class
2. Create a constructor with dependency as arugment
3. Config dependency in Spring config for inejction

```java
class RandomFortuneService implements FortuneService {

    private static final String[] fortunes = new String[]{"Too bad", "Too lucky", "so so"};
    private static final Random rand = new Random();

    @Override
    public String getDailyFortune() {
        return fortunes[rand.nextInt(3)];
    }
}

class CodeCoach implements Coach {
    private final FortuneService fortuneService;

    public CodeCoach(FortuneService fortuneService) {
        this.fortuneService = fortuneService;
    }

    @Override
    public String getDailyWorkout() {
        return "Just LeetCode";
    }

    @Override
    public String getDailyFortune() {
        return "You chance to get the job: " + fortuneService.getDailyFortune();
    }
}
```

```xml
    <bean id="myRandFortune"
          class="note.spring.springdemo.RandomFortuneService">
    </bean>

    <bean id="myCoach"
          class="note.spring.springdemo.CodeCoach">

        <!-- set up constructor injection, referece to the object id will be injected -->
        <constructor-arg ref="myRandFortune"/>
    </bean>
```

What happened behinde the scene?

```java
RandomFortuneService myRandFortune = new RandomFortuneService();

CodeCoach myCoach = new CodeCoach(myRandFortune);
```

***Setter Injection***

1. Create setter method for inejection
2. Config dependency in Spring config for inejction

Notice: property name ***abc***, we need to define a setter method callled ***setAbc***.
Spring will based on property name to find setter method for inejction.

```java
class HappyFortuneService implements FortuneService {
    @Override
    public String getDailyFortune() {
        return "Today is your lucky day!!!";
    }
}

class DanceCoach implements Coach {
    private FortuneService fortuneService;

    @Override
    public String getDailyWorkout() {
        return "Just Dance";
    }

    @Override
    public String getDailyFortune() {
        return fortuneService.getDailyFortune();
    }

    public void setFortuneService(FortuneService fortuneService) {
        System.out.println("setter injection called");
        this.fortuneService = fortuneService;
    }
```

```xml
    <bean id="myFortune"
          class="note.spring.springdemo.HappyFortuneService">
    </bean>

    <bean id="myDanceCoach"
          class="note.spring.springdemo.DanceCoach">
        <!-- setter  injection -->
        <property name="fortuneService" ref="myFortune"/>
    </bean>
```

What happened behinde the scene?

```java
HappyFortuneService myFortune = new HappyFortuneService();

DanceCoach myCoach = new DanceCoach();
myCoach.setFortuneService(myFortune);
```

***Setter injection value***

```java
public class DanceCoach implements Coach {
    private FortuneService fortuneService;

    private String team;

    private String email;

    public DanceCoach() {
        System.out.println("Initialize by spring context");
    }

    @Override
    public String getDailyWorkout() {
        return "Just Dance";
    }

    @Override
    public String getDailyFortune() {
        return fortuneService.getDailyFortune();
    }

    public void setFortuneService(FortuneService fortuneService) {
        System.out.println("setter injection called");
        this.fortuneService = fortuneService;
    }

    public void setTeam(String team) {
        System.out.println("setter injection called: " + team);
        this.team = team;
    }

    public void setEmail(String email) {
        System.out.println("setter injection called: " + email);
        this.email = email;
    }

    public String getTeam() {
        return team;
    }

    public String getEmail() {
        return email;
    }
}
```

Files: sports.properties

```
foo.email=def@bbb.com
```

```xml
    <!-- property loader -->
    <context:property-placeholder location="classpath:sports.properties"/>

    <bean id="myDanceCoach"
          class="note.spring.springdemo.DanceCoach">

        <!-- setter  injection -->
        <property name="fortuneService" ref="myFortune"/>

        <!-- setter injection -->
        <property name="team" value="Class A" />

        <!-- setter injection by property config -->
        <property name="email" value="${foo.email}"/>
    </bean>
```