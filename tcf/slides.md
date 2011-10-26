!SLIDE subsection
# Spring TestContext Framework


!SLIDE incremental
# Overview

* Introduced in Spring 2.5
* Unit and integration testing
* Annotation-driven
* Convention over Configuration
* JUnit & TestNG


!SLIDE incremental small
# Spring & Unit Testing

* POJO-based programming model
* Program to interfaces
* IoC / Dependency Injection
* Out-of-container testability
* Testing mocks/stubs for various APIs: Servlet,  Portlet, JNDI
* General purpose testing utilities
  * `ReflectionTestUtils`
  * `ModelAndViewAssert`


!SLIDE incremental small
# Spring & Integration Testing

* `ApplicationContext` management & caching
* Dependency injection of test instances
* Transactional test management
  * with default rollback semantics
* `SimpleJdbcTestUtils`
* JUnit 3.8 support classes are deprecated as of Spring 3.0/3.1


!SLIDE subsection small
# Core Components


!SLIDE incremental
# `TestContext`

* Tracks context for current test
* Delegates to a `ContextLoader`
* Caches `ApplicationContext`


!SLIDE incremental
# `TestContextManager`

* Manages the `TestContext`
* Signals events to listeners:
  * before: _before-class methods_
  * after: _test instantiation_
  * before: _before methods_
  * after: _after methods_
  * after: _after-class methods_


!SLIDE incremental small
# `TestExecutionListener` SPI

* Reacts to test execution events
  * Receives reference to current `TestContext`
* Out of the box:
  * `DependencyInjectionTestExecutionListener`
  * `DirtiesContextTestExecutionListener`
  * `TransactionalTestExecutionListener`


!SLIDE center small
# `TestExecutionListener`

![Testing-TEL-CD.png](Testing-TEL-CD.png)


!SLIDE center small
# TEL: _Prepare Instance_

![Testing-TEL-SD-prepareTestInstance.png](Testing-TEL-SD-prepareTestInstance.png)


!SLIDE center small
# TEL: _Befores and Afters_

![Testing-TEL-SD-befores-and-afters.png](Testing-TEL-SD-befores-and-afters.png)


!SLIDE incremental small
# `ContextLoader` SPI

* `ContextLoader` & `SmartContextLoader` SPIs
  * Strategy for loading application contexts
* Out of the box:
  * `GenericXmlContextLoader`
  * `GenericPropertiesContextLoader`
  * `AnnotationConfigContextLoader`
  * `DelegatingSmartContextLoader`


!SLIDE center small
# `ContextLoader` 2.5

![Testing-ContextLoader-CD-2.5.png](Testing-ContextLoader-CD-2.5.png)


!SLIDE center small
# Putting it all together

![Testing-TCF-CoreComponents.png](Testing-TCF-CoreComponents.png)


!SLIDE incremental small
# Spring Test Annotations

* Application Contexts
  * `@ContextConfiguration`, `@DirtiesContext`
* `@TestExecutionListeners`
* Dependency Injection
  * `@Autowired`, `@Qualifier`, `@Inject`, ...
* Transactions
  * `@Transactional`, `@TransactionConfiguration`, `@Rollback`, `@BeforeTransaction`, `@AfterTransaction`


!SLIDE incremental
# Spring JUnit Annotations

* Testing Profiles
  * _groups, not bean definition profiles_
  * `@IfProfileValue`, `@ProfileValueSourceConfiguration`
* JUnit extensions
  * `@ExpectedException`, `@Timed`, `@Repeat`


!SLIDE incremental
# Using the TestContext Framework

* Use the `SpringJUnit4ClassRunner` for __JUnit__ 4.5+
* Instrument test class with `TestContextManager` for __TestNG__
* Extend one of the base classes
  * `Abstract(Transactional)[JUnit4|TestNG]SpringContextTests`


!SLIDE small
# Example: Annotated Test Class

	@@@ java
	@RunWith(SpringJUnit4ClassRunner.class)
	@ContextConfiguration
	// defaults to MyTests-context.xml in same package
	@Transactional
	public class MyTests {

	  @Autowired
	  private MyService myService;

	  @BeforeTransaction
	  public void verifyInitialDatabaseState() { … }

	  @Before
	  public void setUpTestDataWithinTransaction() { … }

	  @Test
	  public void myTest() { … }
	}


!SLIDE incremental
# New in Spring 3.1

* @Configuration classes
* Environment profiles
* SmartContextLoader SPI
* Updated context cache key generation

!SLIDE center small
# `ContextLoader` 2.5

![Testing-ContextLoader-CD-2.5.png](Testing-ContextLoader-CD-2.5.png)


!SLIDE center small
# `ContextLoader` 3.1

![Testing-ContextLoader-CD-3.1.png](Testing-ContextLoader-CD-3.1.png)
