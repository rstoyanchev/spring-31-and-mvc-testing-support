!SLIDE subsection transition=scrollDown
# New in Spring 3.1

!SLIDE bullets center
* Support for ...

!SLIDE bullets center
* testing with `@Configuration` classes

!SLIDE bullets center
* and

!SLIDE bullets center
* testing with _environment profiles_

!SLIDE bullets center
* made possible by ...

!SLIDE bullets center
* `SmartContextLoader`

!SLIDE bullets center
* `AnnotationConfigContextLoader`

!SLIDE bullets center
* `DelegatingSmartContextLoader`

!SLIDE bullets center
* updated context cache key generation


!SLIDE incremental
# `SmartContextLoader` SPI

* Supersedes `ContextLoader`
* Strategy for loading application contexts
* From `@Configuration` classes _or_ resource locations
* Supports environment profiles


!SLIDE incremental
# Implementations

* `GenericXmlContextLoader`
* `GenericPropertiesContextLoader`
* `AnnotationConfigContextLoader`
* `DelegatingSmartContextLoader`


!SLIDE center small transition=fade
# `ContextLoader` 2.5
![Testing-ContextLoader-CD-2.5.png](Testing-ContextLoader-CD-2.5.png)

!SLIDE center small transition=fade
# `ContextLoader` 3.1
![Testing-ContextLoader-CD-3.1.png](Testing-ContextLoader-CD-3.1.png)


!SLIDE subsection
# Testing with<br /> @Configuration Classes


!SLIDE
# `@ContextConfiguration`

accepts a new `classes` attribute...

!SLIDE
	@@@ java

	@ContextConfiguration(
		classes={DataAccessConfig.class,
				 ServiceConfig.class})


!SLIDE bullets center

* First, a review with XML config for comparison...

!SLIDE code small
# `OrderServiceTest-context.xml`

	@@@ xml
	<?xml version="1.0" encoding="UTF-8"?>
	<beans ...>

	  <!-- will be injected into OrderServiceTest -->
	  <bean id="orderService"
	      class="com.example.OrderServiceImpl">
	    <!-- set properties, etc. -->
	  </bean>

	  <!-- other beans -->

	</beans>


!SLIDE code small
# `OrderServiceTest.java`

	@@@ java
	package com.example;

	@RunWith(SpringJUnit4ClassRunner.class)
	@ContextConfiguration
	public class OrderServiceTest {

	    @Autowired
	    private OrderService orderService;

	    @Test
	    public void testOrderService() {
	        // test the orderService
	    }
	}


!SLIDE bullets center

* Let's rework that example to use a `@Configuration` class and the new `AnnotationConfigContextLoader`...

!SLIDE code smaller
# `OrderServiceTest.java`

	@@@ java
	@RunWith(SpringJUnit4ClassRunner.class)


	public class OrderServiceTest {












	  @Autowired
	  private OrderService orderService;
	  // @Test methods ...
	}


!SLIDE code smaller
# `OrderServiceTest.java`

	@@@ java
	@RunWith(SpringJUnit4ClassRunner.class)
	@ContextConfiguration(
		loader=AnnotationConfigContextLoader.class)
	public class OrderServiceTest {












	  @Autowired
	  private OrderService orderService;
	  // @Test methods ...
	}


!SLIDE code smaller
# `OrderServiceTest.java`

	@@@ java
	@RunWith(SpringJUnit4ClassRunner.class)
	@ContextConfiguration
	public class OrderServiceTest {












	  @Autowired
	  private OrderService orderService;
	  // @Test methods ...
	}


!SLIDE code smaller
# `OrderServiceTest.java`

	@@@ java
	@RunWith(SpringJUnit4ClassRunner.class)
	@ContextConfiguration
	public class OrderServiceTest {

	  @Configuration
	  static class Config {







	  }

	  @Autowired
	  private OrderService orderService;
	  // @Test methods ...
	}


!SLIDE code smaller
# `OrderServiceTest.java`

	@@@ java
	@RunWith(SpringJUnit4ClassRunner.class)
	@ContextConfiguration
	public class OrderServiceTest {

	  @Configuration
	  static class Config {

	    @Bean // will be injected into OrderServiceTest
	    public OrderService orderService() {
	      OrderService orderService = new OrderServiceImpl();
	      // set properties, etc.
	      return orderService;
	    }
	  }

	  @Autowired
	  private OrderService orderService;
	  // @Test methods ...
	}

!SLIDE incremental
# What's changed?

* No XML
* Bean definitions are converted to Java
  * using `@Configuration` and `@Bean`
* Otherwise, the test remains unchanged

!SLIDE bullets center

* But what if we don't want a static inner `@Configuration` class?

!SLIDE bullets center

* Just externalize the config...

!SLIDE code smaller
# `OrderServiceConfig.java`

	@@@ java
	@Configuration
	public class OrderServiceConfig {

	  @Bean // will be injected into OrderServiceTest
	  public OrderService orderService() {
	    OrderService orderService = new OrderServiceImpl();
	    // set properties, etc.
	    return orderService;
	  }
	}

!SLIDE bullets center

* And reference the config classes in `@ContextConfiguration`...

!SLIDE code smaller
# `OrderServiceConfig.java`

	@@@ java
	@RunWith(SpringJUnit4ClassRunner.class)
	@ContextConfiguration(classes=OrderServiceConfig.class)
	public class OrderServiceTest {

	  @Autowired
	  private OrderService orderService;

	  @Test
	  public void testOrderService() {
	    // test the orderService
	  }
	}


!SLIDE incremental
# @Configuration + XML

* __Q__: How can we combine `@Configuration` classes with XML config?
* __A__: Choose one as the _entry point_.
* _That's how it works in production anyway_


!SLIDE incremental
# Importing Configuration

* In XML:
  * include `@Configuration` classes via component scanning
  * or define them as normal Spring beans
* In an `@Configuration` class:
  * use `@ImportResource` to import XML config files


!SLIDE subsection
# Testing with<br /> Environment Profiles


!SLIDE incremental
# @ActiveProfiles

To activate _bean definition profiles_ in tests...

* Annotate a test class with `@ActiveProfiles`
* Supply a list of profiles to be activated for the test
* Can be used with `@Configuration` classes and XML config

!SLIDE bullets center
* Let's look at an example with XML config...

!SLIDE code smaller
# `app-config.xml` (1/2)

	@@@ xml
	<beans ... >

		<bean id="transferService"
			class="com.example.DefaultTransferService">
			<constructor-arg ref="accountRepository"/>
			<constructor-arg ref="feePolicy"/>
		</bean>

		<bean id="accountRepository"
			class="com.example.JdbcAccountRepository">
			<constructor-arg ref="dataSource"/>
		</bean>

		<bean id="feePolicy"
			class="com.example.ZeroFeePolicy"/>
			
		<!-- dataSource -->

	</beans>


!SLIDE code smaller
# `app-config.xml` (2/2)

	@@@ xml
	<beans ... >

		<!-- transferService, accountRepository, feePolicy -->















	</beans>


!SLIDE code smaller
# `app-config.xml` (2/2)

	@@@ xml
	<beans ... >

		<!-- transferService, accountRepository, feePolicy -->

		<beans profile="production">
			<jee:jndi-lookup id="dataSource"
				jndi-name="java:comp/env/jdbc/datasource"/>
		</beans>










	</beans>


!SLIDE code smaller
# `app-config.xml` (2/2)

	@@@ xml
	<beans ... >

		<!-- transferService, accountRepository, feePolicy -->

		<beans profile="production">
			<jee:jndi-lookup id="dataSource"
				jndi-name="java:comp/env/jdbc/datasource"/>
		</beans>

		<beans profile="dev">
			<jdbc:embedded-database id="dataSource">
				<jdbc:script
					location="classpath:schema.sql"/>
				<jdbc:script
					location="classpath:test-data.sql"/>
			</jdbc:embedded-database>
		</beans>

	</beans>


!SLIDE code smaller
# `TransferServiceTest.java`

	@@@ java
	@RunWith(SpringJUnit4ClassRunner.class)


	public class TransferServiceTest {

	    @Autowired
	    private TransferService transferService;

	    @Test
	    public void testTransferService() {
	        // test the transferService
	    }
	}


!SLIDE code smaller
# `TransferServiceTest.java`

	@@@ java
	@RunWith(SpringJUnit4ClassRunner.class)
	@ContextConfiguration("/app-config.xml")

	public class TransferServiceTest {

	    @Autowired
	    private TransferService transferService;

	    @Test
	    public void testTransferService() {
	        // test the transferService
	    }
	}


!SLIDE code smaller
# `TransferServiceTest.java`

	@@@ java
	@RunWith(SpringJUnit4ClassRunner.class)
	@ContextConfiguration("/app-config.xml")
	@ActiveProfiles("dev")
	public class TransferServiceTest {

	    @Autowired
	    private TransferService transferService;

	    @Test
	    public void testTransferService() {
	        // test the transferService
	    }
	}


!SLIDE bullets center
* And now an example with `@Configuration` classes

!SLIDE code smaller
# `TransferServiceConfig.java`

	@@@ java
	@Configuration
	public class TransferServiceConfig {

	  @Autowired DataSource dataSource;

	  @Bean
	  public TransferService transferService() {
	    return new DefaultTransferService(accountRepository(),
	      feePolicy());
	  }

	  @Bean
	  public AccountRepository accountRepository() {
	    return new JdbcAccountRepository(dataSource);
	  }

	  @Bean
	  public FeePolicy feePolicy() {
	    return new ZeroFeePolicy();
	  }
	}


!SLIDE code smaller
# `JndiDataConfig.java`

	@@@ java
	@Configuration
	@Profile("production")
	public class JndiDataConfig {

	  @Bean
	  public DataSource dataSource() throws Exception {
	    Context ctx = new InitialContext();
	    return (DataSource)
	      ctx.lookup("java:comp/env/jdbc/datasource");
	  }
	}


!SLIDE code smaller
# `StandaloneDataConfig.java`

	@@@ java
	@Configuration
	@Profile("dev")
	public class StandaloneDataConfig {

	  @Bean
	  public DataSource dataSource() {
	    return new EmbeddedDatabaseBuilder()
	      .setType(EmbeddedDatabaseType.HSQL)
	      .addScript("classpath:schema.sql")
	      .addScript("classpath:test-data.sql")
	      .build();
	  }
	}


!SLIDE bullets center
* And finally the test class...

!SLIDE code smaller
# `TransferServiceTest.java`

	@@@ java
	package com.bank.service;

	@RunWith(SpringJUnit4ClassRunner.class)






	public class TransferServiceTest {

	    @Autowired
	    private TransferService transferService;

	    @Test
	    public void testTransferService() {
	        // test the transferService
	    }
	}


!SLIDE code smaller
# `TransferServiceTest.java`

	@@@ java
	package com.bank.service;

	@RunWith(SpringJUnit4ClassRunner.class)
	@ContextConfiguration(
	    classes={
	        TransferServiceConfig.class,
	        StandaloneDataConfig.class,
	        JndiDataConfig.class})

	public class TransferServiceTest {

	    @Autowired
	    private TransferService transferService;

	    @Test
	    public void testTransferService() {
	        // test the transferService
	    }
	}


!SLIDE code smaller
# `TransferServiceTest.java`

	@@@ java
	package com.bank.service;

	@RunWith(SpringJUnit4ClassRunner.class)
	@ContextConfiguration(
	    classes={
	        TransferServiceConfig.class,
	        StandaloneDataConfig.class,
	        JndiDataConfig.class})
	@ActiveProfiles("dev")
	public class TransferServiceTest {

	    @Autowired
	    private TransferService transferService;

	    @Test
	    public void testTransferService() {
	        // test the transferService
	    }
	}


!SLIDE incremental small
# Active Profile Inheritance

* `@ActiveProfiles` supports _inheritance_ as well
* Via the `inheritProfiles` attribute
* See Javadoc for an example

!SLIDE subsection
# ApplicationContext Caching

!SLIDE bullets center
* Until Spring 3.1

!SLIDE bullets center
* application contexts were cached

!SLIDE bullets center
* but using only resource locations for the key.

!SLIDE bullets center
* Now there are different requirements...

!SLIDE incremental small
# New Key Generation Algorithm

The context cache key generation algorithm has been updated to include...

* __locations__ _(from `@ContextConfiguration`)_
* __classes__ _(from `@ContextConfiguration`)_
* __contextLoader__ _(from `@ContextConfiguration`)_
* __activeProfiles__ _(from `@ActiveProfiles`)_


!SLIDE incremental small transition=scrollUp
# Summary

* The _Spring TestContext Framework_ simplifies integration testing of Spring-based applications
* Spring 3.1 provides first-class testing support for:
  * `@Configuration` classes
  * Environment profiles
* See the <a href="http://blog.springsource.com/2011/06/21/spring-3-1-m2-testing-with-configuration-classes-and-profiles/">Testing with @Configuration Classes and Profiles</a> blog for further insight

