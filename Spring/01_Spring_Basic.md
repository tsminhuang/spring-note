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

## Spring Bean Scopes

+ singleton: create a single instace in one container
+ prototype : create new bean for each container request
+ request: HTTP web request
+ session: HTTP web session
+ global-session: gobal HTPP web session

### Singleton

+ Singleton
  + defaul scope
  + cached in memory
  + share reference to the same bean

```xml
    <bean id="myCoach"
          class="note.spring.springdemo.service.impl.TrackCoach"
          scope="singleton">

        <!-- set up constructor injection -->
        <constructor-arg ref="myFortune"/>
    </bean>

    <bean id="myCodeCoach"
          class="note.spring.springdemo.service.impl.CodeCoach"
          scope="prototype">

        <!-- set up constructor injection -->
        <constructor-arg ref="myRandFortune"/>
    </bean>
```

```java
public class BeanScopeDemoApp {
    public static void main(String[] argv) {
        // load spring configuration config
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("beanScope-applicationContext.xml");

        // singleton bean
        Coach theCoach = context.getBean("myCoach", Coach.class);
        Coach alphaCoach = context.getBean("myCoach", Coach.class);

        boolean isSameCoach = (theCoach == alphaCoach);
        System.out.println("singleton bean");
        System.out.println("The coach is the same: " + isSameCoach);
        System.out.println("the theCoach address: " + theCoach);
        System.out.println("the alphaCoach address: " + alphaCoach);

        // prototype bean
        theCoach = context.getBean("myCodeCoach", Coach.class);
        alphaCoach = context.getBean("myCodeCoach", Coach.class);
        isSameCoach = (theCoach == alphaCoach);
        System.out.println("prototype bean");
        System.out.println("The coach is the same: " + isSameCoach);
        System.out.println("the theCoach address: " + theCoach);
        System.out.println("the alphaCoach address: " + alphaCoach);

        // close the context
        context.close();
    }
}
```

### Bean Life Cycle

Container Start -> Bean creation -> Dependenies Injected -> Internal Spring Process -> Your Customer init Method

+ post construction:
  + calling customize business logic
  + setting up handles to resource (db, sockes, files etc)

Container is shut donw -> Your Custom Destroy -> Stop

+ pre destroy:
  + calling customize business logic
  + clean up handles to resource (db, sockes, files etc)

+ can any access level (since it use reflection)
+ **must be no arg method**
+ can have return type but will not use in Spring
+ Spring did not full manage the lifecyce of prototype scope: **need to implement CustomBeanPostProcessor** and prototype scope bean need to implement **DisposableBean** interface

```xml
    <bean id="myCoach"
          class="note.spring.springdemo.service.impl.TrackCoach"
          init-method="MyInit"
          destroy-method="MyCleanUp">

        <!-- set up constructor injection -->
        <constructor-arg ref="myFortune"/>
    </bean>

    <bean id="myCodeCoach"
          class="note.spring.springdemo.service.impl.CodeCoach"
          scope="prototype"
          init-method="MyInit"
          destroy-method="destroy">

        <!-- set up constructor injection -->
        <constructor-arg ref="myRandFortune"/>
    </bean>
```

```java
public class MyCustomBeanProcessor implements BeanPostProcessor, BeanFactoryAware, DisposableBean {

    private BeanFactory beanFactory;

    private final List<Object> prototypeBeans = new LinkedList<>();

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {

        // after start up, keep track of the prototype scoped beans.
        // we will need to know who they are for later destruction

        if (beanFactory.isPrototype(beanName)) {
            synchronized (prototypeBeans) {
                prototypeBeans.add(bean);
            }
        }

        return bean;
    }


    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
    }


    @Override
    public void destroy() throws Exception {

        // loop through the prototype beans and call the destroy() method on each one

        synchronized (prototypeBeans) {

            for (Object bean : prototypeBeans) {

                if (bean instanceof DisposableBean) {
                    DisposableBean disposable = (DisposableBean) bean;
                    try {
                        disposable.destroy();
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }

            prototypeBeans.clear();
        }

    }
}
```

## Java Annotation

+ Label/Marker for Java classes
+ Meta-data about the class
+ Process at compile time or run-time for special process

### IoC

1. enable component scale in Spring config file
2. add annotation like @Component to class
3. retrive bean from Spring container

file: applicationContent.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd">

    <!--  add component scan  -->
    <context:component-scan base-package="note.spring.springdemo"/>

</beans>
```

```java
@Component("thatSillyCoach")
public class TennisCoach implements Coach {
    @Override
    public String getDailyWorkout() {
        return "Run 5k";
    }
}
```

```java
public class AnnotationDemoApp {

    public static void main(String[] argv) {
        // read spring config
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContent.xml");

        // retrieve bean from spring
        System.out.println("=============================");
        Coach theCoach = context.getBean("thatSillyCoach", Coach.class);
        System.out.println(theCoach.getDailyWorkout());
    }
}
```

### Bean name rule

+ explicity give name for the bean
+ implicity name rule: use class name but replace first letter to lowercase
  + If class name is have more then one upper case in class name the bean name will the same: RESTFortuneService (class name) -> RESTFortuneService (bean name)

```java
@Component("thatSillyCoach")
public class TennisCoach implements Coach {
}

// bean name will be codeCoach
@Component()
public class CodeCoach implements Coach {
}

// bean name will be RESTFortuneService
@Component
public class RESTFortuneService implements FortuneService {
}

```

### @Autowired for DI

The dependecy to inejct must also a bean (aka ineject class should also annotated with @Component annotatation)

Constructor Injection

```java
@Component
public class TennisCoach implements Coach {
    private FortuneService theFortuneService;

    // After Spring 4.3 annotation for @Autowired can ignore
    // if we only have 1 constructor
    @Autowired
    public TennisCoach(FortuneService fortuneService) {
        System.out.println(">> TennisCoach: constructor injection");
        theFortuneService = fortuneService;
    }
}
```

Setter Injection

```java
@Component
public class CodeCoach implements Coach {
    private FortuneService theFortuneService;
    @Autowired
    private void setFortuneService(FortuneService theFortuneService) {
        System.out.println(">> CodeCoach setter injection to setFortuneService");
        fortuneService = theFortuneService;
    }
}
```

Filed Injection (done by Java reflection)

```java
@Component
public class DanceCoach implements Coach {

    @Autowired
    private FortuneService fortuneService;
}
```

### Property for field value

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd">

    <!--  add component scan  -->
    <context:component-scan base-package="note.spring.springdemo"/>

    <!--  add property config  -->
    <context:property-placeholder location="classpath:sport.properties"/>

</beans>
```

file: sport.properties"
```
foo.email=abc@def.com
foo.team=Class B
```

```java
@Component
public class DanceCoach implements Coach {
    @Value("${foo.email}")
    private String email;

    @Value("${foo.team}")
    private String team;
}
```

### @Qualifier to avoid ambiguity

If we have muliple class all implement same interface how does spring how which should inject?

```java
public interface FortuneService {
    String getFortune();
}

@Component
public class HappyFortuneService implements FortuneService {
}

@Component
public class RandomFortuneService implements FortuneService {
}

@Component
public class RESTFortuneService implements FortuneService {
}
```

**@Qualifier** can solved ambiguity

Constructor Inejection: **@Qualifier** need to put in arguement

```java
@Component("thatSillyCoach")
// We don't need to add scope annotation since default is singleton
public class TennisCoach implements Coach {

    private final FortuneService theFortuneService;

    // After Spring 4.3 annotation for @Autowired can ignore
    // if we only have 1 constructor
    @Autowired
    public TennisCoach(@Qualifier("happyFortuneService") FortuneService fortuneService) {
        System.out.println(">> TennisCoach: constructor injection");
        theFortuneService = fortuneService;
    }
```

Setter Inject: can just put on method name

```java
@Component
@Scope("prototype")
public class CodeCoach implements Coach, DisposableBean {
    private FortuneService fortuneService;
    @Autowired
    @Qualifier("randomFortuneService")
    private void setFortuneService(FortuneService theFortuneService) {
        System.out.println(">> CodeCoach setter injection to setFortuneService");
        fortuneService = theFortuneService;
    }
```

Field Injection: can put on varable name

```java
@Component
public class DanceCoach implements Coach {

    @Autowired
    @Qualifier("RESTFortuneService")
    private FortuneService fortuneService;
}
```

### Bean Scope and Life cycle

+ @Scope
+ @PostConstruct
+ @PreDestroy

```java
@Component("thatSillyCoach")
// We don't need to add scope annotation since default is singleton
//@Scope("singleton")
public class TennisCoach implements Coach {
    @PostConstruct
    private void myInit() {
        System.out.println(">> TennisCoach: PostConstruct called ");
    }

    @PreDestroy
    public void myCleanUp() {
        System.out.println(">> TennisCoach: PreDestroy called ");
    }
}

@Component
@Scope("prototype")
public class CodeCoach implements Coach, DisposableBean {
    @PostConstruct
    private void myInit() {
        System.out.println(">> CodeCoach: PostConstruct called ");
    }

    // prototype scope will not call method with PreDestroy annotation
    //@PreDestroy
    public void myCleanUp() {
        System.out.println(">> CodeCoach: PreDestroy called ");
    }

    @Override
    public void destroy() throws Exception {
        myCleanUp();
    }
}
```

## Java Code Configuration

### @Configuration, @Bean and @PropertySource

***Configuration***

1. create config class with @Configuration annotation
2. add @ComponentScan to class (optional)
3. read Spring Java Configuration class
4. retrive Bean from Spring Container

***Bean***

This is useful to wrap 3rd-party class as @Bean to inject our service.

1. define @Bean to expose method
2. inject Bean dependencies
3. read Spring Java configuration class
4. retriveveBean from Spring Container


***PropertySource***

1. create property file
2. read property file by @PropertySource in Java config class
3. use @Value annotation to retrive value

```java
@Configuration
//@ComponentScan("note.spring.springdemo") optional
@PropertySource("classpath:sport.properties")
public class JobConfig {

    @Bean
    public FortuneService sadFortuneService() {
        return new SadFortuneService();
    }

    @Bean
    public Coach jobCoach() {
        return new JobCoach(sadFortuneService());
    }
}

public class JobJavaConfigDemoApp {
    public static void main(String[] argv) {
        // read spring config
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(JobConfig.class);

        // retrieve singleton bean from spring
        System.out.println("=============================");
        JobCoach theCoach = context.getBean("jobCoach", JobCoach.class);
        System.out.println(theCoach.getDailyWorkout());
        // setter injection for fortune service
        System.out.println(theCoach.getDailyFortune());

        System.out.println(theCoach.getEmail());
        System.out.println(theCoach.getTeam());

        // close context
        context.close();
    }
}
```