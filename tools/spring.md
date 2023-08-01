---

mindmap-plugin: basic

---

# Spring

## What?
- A Dependency Injection (DI) framework
- Inversion of Control (IoC) principle: Spring framework controls the app & its dependencies instead of the opposite

## Concepts
- Spring context
   - A container in memory that creates, wires and manages application components
- Beans
   - Application components (objects) contained in Spring context
   - Injected to the application as properties
- Configuration (class)
   - Used to instruct Spring to do specific actions
      - Adding beans to Spring context
- Repository/Data access objects (DAO)
   - Objects working directly with a database
- Bean scope
   - Singleton
      - Default scope
      - Each bean (defined by (class, bean name/id) pair) is associated with (only) 1 instance
      - Each time a bean is requested, same instance is returned
      - Instance should be immutable, or made thread-safe (not recommended due to performance implication)
      - Initialization
         - Eager
            - Default behavior
            - Instance is created when Spring context is created
         - Lazy
            - Instance is created when the bean is requested
   - Prototype
      - Each bean is associated with a type
      - Each time a bean is requested, a new instance is created and returned
- Spring AOP
   - AOP = #aspect-oriented-programming
      - Aspect = the way a framework intercepts method calls
         - Adding pre/post processing logic
         - Intercept method parameters
         - Intercept return value
      - Pros: logic decoupling
      - Terms
         - *aspect*: what code to execute when a method is called
         - *advice*: when
         - *pointcut*: which method to intercept
         - *target object*: object whose methods intercepted by an aspect
   - #weaving
      - Approach used by Spring to support AOP
      - Target object must be a bean in the Spring context
      - Returned object is a proxy/decorated object by Spring instead of the original one
   - Aspects execution chain
      - A method can be intercepted by many aspects
      - Spring does not guarantee execution order by default
      - Can use `@Order` annotation to specify execution order

## Hands-on
- Adding beans to Spring context
   - How?
      - Mark method providing the component with `@Bean`, method is defined in configuration class
      - Mark class with Stereotype annotations, then use `@ComponentScan` over the configuration class
         - `@Component`: generic annotation
         - `@Service`: classes that implement use cases
         - `@Repository`: classes that manage data persistence
      - Programmatically with `context.registerBean`
   - General guide on what to add
      - Dependency classes
      - Classes that have dependencies in Spring context
- Establishing beans relationship
   - Wiring with `@Bean`

      -
        ```java
        @Bean
        Y y(X x) {}  // X is another bean
        ```

   - Auto-wiring with `@Autowired`
      - Injecting beans through fields

         -
           ```java
           @Component
           public class Y {
           @Autowired private X x;
           }
           ```

         - Cons
            - Fields annotated with `@Autowired` can't be made final
            - Hard to write unit tests
      - Injecting beans through constructors

         -
           ```java
           @Component
           public class Y {
           private final X x;
           
           @Autowired
           public Y(X x) { this.x = x }
           }
           ```

         - Recommended prod approach
         - No need annotation if class only has 1 constructor
      - Injecting beans through setters
         - Same cons with injecting through fields
         - Not recommended
- Dealing with multiple beans
   - Resolving bean by parameter name

      -
        ```java
        @Bean x1();
        @Bean x2();
        @Bean y(X x1);  // matches x1 Bean by param name
        ```

      - Not recommended, since name can be changed accidently
   - Resolving bean by `@Qualifier`

      -
        ```java
        @Bean x1();
        @Bean x2();
        @Bean y(@Qualifier("x2") X x);
        ```

      - Recommended

## References
- #book-spring-in-action-6ed
- #book-spring-start-here

## Ecosystem
- Spring Core
   - Spring Data Access
- [[spring-boot]]
   - Purpose: create prod-ready Spring app with minimal configurations
   - Features
      - Simplified project creation
      - Simplified dependency management
         - Grouping dependencies per capability (web, db, security, etc.)
         - As a result, we just specify the capability-oriented group of dependencies we want to use
      - Autoconfiguration
- Spring Data
- Spring Security
- Spring Cloud
- Spring Batch
- Spring MVC
   - Components
      - Dispatcher servlet
      - Handler mapping
      - Controller
      - View resolver