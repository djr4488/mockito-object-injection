# Mockito Object Injection

Inject Strings (or other objects) into your `@InjectMocks` targets [objects under test] without booting a Spring, Weld, CDI, Arquillian, EJB, or other container. Super lightweight and easy to use. Skip straight to Examples and Usage if you know what you're looking for.

## Problem

Take this Spring Controller (or if you're using the far superior and modern CDI framework, imagine this is `@AppplicationScoped`)

```
@Controller
public class MyController {
 @Value("securityEnabled")
 private Boolean securityEnabled;
 @Autowired
 private Authenticator auther;
 @Autowired
 private Logger log;

 public void doSomething() {
  if (securityEnabled) {
    auther.something();
  } else {
    log.warn("sec disabled");
  }
 }
}
```

If you wanted to write a _true unit test*_ with no external dependencies, you'd probably want to use Mockito mock your dependencies:

```
@ExtendWith({ MockitoExtension.class })
public class MyControllerTest {
 @InjectMocks
 private MyController myController;
 @Mock
 private Logger log;
 @Mock
 private Authenticator auther;
  
  public void testDoSomething() throws Exception {
   myController.doSomething();
   // results in NPE
  }
 }
```

## Examples

This JUnit5 extension allows you to arbitrarily set any field on your `@InjectMocks` [class under test] target. The injections happen _very late_; they happen when you call any non-private method on the class under test.


```
import com.github.exabrial.junit5.injectmap.InjectMap;
import com.github.exabrial.junit5.injectmap.InjectMapExtension;

@ExtendWith({ MockitoExtension.class, InjectMapExtension.class })
public class MyControllerTest {
 @InjectMocks
 private MyController myController;
 @Mock
 private Logger log;
 @Mock
 private Authenticator auther;
 @InjectMap
 private Map<String, Object> injectMap = new HashMap<>();
 
 @BeforeEach
 public void beforeEach() throws Exception {
  injectMap.put("securityEnabled", Boolean.TRUE);
 }

 @AfterEach
 public void afterEach() throws Exception {
  injectMap.clear();
 }
  
 public void testDoSomething_secEnabled() throws Exception {
  myController.doSomething();
  // wahoo no NPE! Test the "if then" half of the branch
 }
  
 public void testDoSomething_secDisabled() throws Exception {
  injectMap.put("securityEnabled", Boolean.FALSE);
  myController.doSomething();
  // wahoo no NPE! Test the "if else" half of branch
 }
}
```

## License

All files are licensed Apache Source License 2.0. Please consider contributing any improvements you make back to the project.

## Usage

Maven Coordinates:

```
<dependency>
 <groupId>org.junit.jupiter</groupId>
 <artifactId>junit-jupiter-api</artifactId>
 <version>5.2.0</version>
 <scope>test</scope>
</dependency>
<dependency>
 <groupId>org.mockito</groupId>
 <artifactId>mockito-core</artifactId>
 <version>2.22.0</version>
 <scope>test</scope>
</dependency>
<dependency>
 <groupId>com.github.exabrial</groupId>
 <artifactId>mockito-object-injection</artifactId>
 <version>1.0.0</version>
 <scope>test</scope>
</dependency>
```

## Philosophy Time

* _A true unit test means testing one unit of code, so firing up an Arquillian, Spring, or CDI container means you are probably not testing just one thing. In Java, a class is generally considered the smallest testable unit of code testable, but there are also static methods and lambdas which might also fall into this category._

Well... A lot of folks on the interwebs at this point will sigh loudly, rock back in their rocking chair, and hike up their trousers and yammer: _"Well that kids is why we use CONSTRUCTOR INJECTION. You gootta expose that Boolean as a CONSTRUCTOOOR variable!!!"_ then proceed to continue to yell at the kids on their lawn. 

While I just finished mocking said people (pardon the pun), there are a lot of well-defined benefits to constructor injection that I won't go into here that are worth considering. I don't like boilerplate code however, and I don't like generated code. So for me, constructor injection, despite it's advantages, brings a lot of verbosity and repetition to the table. The right answer for YOUR organization on whether to use constructor versus field injection as convention depends highly on said organization's ability to follow a plan, to arbitrate conflict resolution, team cohesiveness, mutual respect, a non HIPPO decision process, and focus on business objectives... most of which are soft-skills which are outside the scope of this project. The goal of this project is to help you get to 100% branch coverage with with as little friction from your tech stack and leave the debate to the philosophers. Good luck, test your classes, and write great code!
