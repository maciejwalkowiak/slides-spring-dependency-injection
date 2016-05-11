title: Dependency Injection in Spring
author:
  name: Maciej Walkowiak
  twitter: maciejwalkowiak
output: basic.html
controls: true
theme: sjaakvandenberg/cleaver-light

--

# Dependency injection in <span class="spring">Spring</spring>

--

### Injection types in Spring

- setter
- field
- constructor

--

### Setter Injection

- extremely popular in the age of XML configuration

```xml
<bean id="myService" class="com.foo.bar.MyService">
	<property name="myComponent" ref="myComopnent"/>
</bean>
```
--
### Setter Injection

- using `@Autowired`:

```java
class MyService {
	private MyComponent myComponent;

	@Autowired
	public void setMyComponent(MyComponent myComponent) {
		this.myComponent = myComponent;
	}
}
```
--
### Setter Injection

- by some, used to inject optional dependencies
- today almost never used

--
# Field Injection

--
### Field Injection

```java
@Service
class MyService {
    @Autowired
    private MyComponent myComponent;

    @Override
    public List<String> foo() {
        final List<String> bar = myComponent.findBar();
        // ...
        return foo;
    }
}
```

--
### Field Injection

- the most popular
- the quickest to use
- uses Spring reflection magic to inject properties

--
### Field Injection

- the most popular
- the quickest to use
- uses Spring reflection magic to inject properties
- **harmful**

--
### What's wrong with this?

```java
@Service
class MyService {
    @Autowired
    private MyComponent myComponent;

    @Override
    public List<String> foo() {
        final List<String> bar = myComponent.findBar();
        // ...
        return foo;
    }
}
```
--

### What's wrong with this?

```java
class MyServiceTest {
    private MyService myService = new MyService();

    @Test
    public void testMethod() {
        myService.foo(); // NullPointerException!
    }
}
```
--
### Field Injection sins

- lets create object in **invalid** state
- difficult to use outside of Spring context (unit tests)

--
### What's wrong with this?

```java
@Component
class ComponentA {
    @Inject
    private ComponentB componentB;
}

@Component
class ComponentB {
    @Inject
    private ComponentA componentA;
}
```

--
### Field Injection sins

- lets create object in invalid state
- difficult to use outside of Spring context (unit tests)
- **allows to create circular dependencies**

--
### How many dependencies does this class have?

```java
@Controller
public class FooController extends AbstractBaseController {
    @Inject
    private MyService myService;

    @RequestMapping(method = RequestMethod.POST)
    public List<String> findFoo() {
		return myService.foo();
    }
}
```

--
### Field Injection sins

- lets create object in invalid state
- difficult to use outside of Spring context (unit tests)
- allows to create circular dependencies
- **hides dependencies**

--
### What's wrong with this?

```java
@Component
class MyComponent {
    @Autowired
    private MyCollaborator collaborator;
    
    String foo() {
       return collaborator.bar();
    }
    
    void someMethod(MyCollaborator collaborator) {
        this.collaborator = collaborator;
    }
}
```
--
### Field Injection sins

- lets create object in invalid state
- difficult to use outside of Spring context (unit tests)
- allows to create circular dependencies
- hides dependencies
- **allows overwriting dependencies**

--
# Constructor Injection

--
### Constructor Injection
#### More code :(

```java
@Component
class MyComponent {
	private MyCollaborator collaborator;
 
	@Autowired
	MyComponent(MyCollaborator collaborator) {
		this.collaborator = collaborator;
	}
}
```

--
### Constructor Injection
#### but fields can be final!

```java
@Component
class MyComponent {
	private final MyCollaborator collaborator;
 
	@Autowired
	MyComponent(MyCollaborator collaborator) {
		this.collaborator = collaborator;
	}
}
```

--
### Constructor Injection
#### and dependencies can be verified

```java
@Component
class MyComponent {
	private final MyCollaborator collaborator;
 
	@Autowired
	MyComponent(MyCollaborator collaborator) {
		Assert.notNull(collaborator);
		this.collaborator = collaborator;
	}
}
```

--
### Constructor Injection

- object is always in valid state
- simple to unit test
- simple to extract to non Spring application
- does not hide dependencies
- prevents from creating circular dependencies
- **more code** but safer and easier to maintain

--
### Constructor Injection howto
#### Required dependencies

```java
@Component
class MyComponent {
    private final MyCollaborator collaborator;
      
    @Autowired
    MyComponent(MyCollaborator collaborator) {
        this.collaborator = collaborator;
    }
}
```

--
### Constructor Injection howto
#### Optional dependencies

```java
@Component
class MyComponent {
    private final MyCollaborator collaborator;
    private final Optional<MyRepository> myRepository;
      
    @Autowired
    MyComponent(MyCollaborator collaborator, 
    	        Optional<MyRepository> myRepository) {
        this.collaborator = collaborator;
        this.myRepository = myRepository;
    }
}
```

--
### Constructor Injection howto
#### Injecting properties

```java
@Component
class MyComponent {
    private final MyCollaborator collaborator;
    private final Optional<MyRepository> myRepository;
    private final String myProperty;
    @Autowired
    MyComponent(MyCollaborator collaborator, 
    			Optional<MyRepository> myRepository
    	        @Value("${myProperty}") String myProperty) {
        ...
        this.myProperty = myProperty;
    }
}
```

--
<img src="http://i.giphy.com/U8bDgsXcnIEFy.gif" style="">