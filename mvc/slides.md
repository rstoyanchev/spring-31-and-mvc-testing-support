!SLIDE subsection
# Spring MVC Test Support

!SLIDE
# How Do To Test an `@Controller`?

!SLIDE smaller

    @@@ java

    @Controller
    @RequestMapping("/accounts")
    public class AccountController {

     // ...

        @ModelAttribute
        public Account getAccount(String number) {
            return this.accountManager.getAccount(number);
        }

        @RequestMapping(method = RequestMethod.POST)
        public String save(@Valid Account account, 
                            BindingResult result) {
            if (result.hasErrors()) {
                return "accounts/edit";
            }
            this.accountManager.saveOrUpdate(account);
            return "redirect:accounts";
        }

    }

!SLIDE bullets incremental
# Unit Test?

* Create controller instance
* Mock or stub services & repositories

!SLIDE smaller
# Example 

    @@@ java

    @Test
    public void testSave() {

        Account account = new Account();
        BindingResult result = 
            new BeanPropertyBindingResult(account, "account");






        

        


    }

!SLIDE smaller
# Example 

    @@@ java

    @Test
    public void testSave() {

        Account account = new Account();
        BindingResult result = 
            new BeanPropertyBindingResult(account, "account");

        AccountManager mgr = createMock(AccountManager.class);
        mgr.saveOrUpdate(account);
        replay(mgr);


        

        


    }

!SLIDE smaller
# Example 

    @@@ java

    @Test
    public void testSave() {

        Account account = new Account();
        BindingResult result = 
            new BeanPropertyBindingResult(account, "account");

        AccountManager mgr = createMock(AccountManager.class);
        mgr.saveOrUpdate(account);
        replay(mgr);

        AccountController contrlr = new AccountController(mgr);
        
        String view = contrlr.save(account, result);
        


    }

!SLIDE smaller
# Example

    @@@ java

    @Test
    public void testSave() {

        Account account = new Account();
        BindingResult result = 
            new BeanPropertyBindingResult(account, "account");

        AccountManager mgr = createMock(AccountManager.class);
        mgr.saveOrUpdate(account);
        replay(mgr);

        AccountController contrlr = new AccountController(mgr);
        
        String view = contrlr.save(account, result);
        
        assertEquals("redirect:accounts", view);
        verify(mgr);
    }

!SLIDE smaller
# Example With Failure

    @@@ java

    @Test
    public void testSave() {

        Account account = new Account();
        BindingResult result = 
            new BeanPropertyBindingResult(account, "account");

        result.reject("error.code", "default message")





        

        


    }

!SLIDE smaller
# Example With Failure

    @@@ java

    @Test
    public void testSave() {

        Account account = new Account();
        BindingResult result = 
            new BeanPropertyBindingResult(account, "account");

        result.reject("error.code", "default message")

        AccountManager mgr = createMock(AccountManager.class);
        replay(mgr);


        

        


    }

!SLIDE smaller
# Example With Failure

    @@@ java

    @Test
    public void testSave() {

        Account account = new Account();
        BindingResult result = 
            new BeanPropertyBindingResult(account, "account");

        result.reject("error.code", "default message")

        AccountManager mgr = createMock(AccountManager.class);
        replay(mgr);

        AccountController contrlr = new AccountController(mgr);
        
        String view = contrlr.save(account, result);
        
        assertEquals("accounts/edit", view);
        verify(mgr);
    }

!SLIDE incremental bullets
# Not Fully Tested

* Request mappings
* Binding errors
* Type conversion 
* Etc.

!SLIDE incremental bullets
# Integration Test?

* Selenium
* JWebUnit
* etc.

!SLIDE incremental bullets
# It Requires..

* A running servlet container
* More time to execute
* More effort to maintain
* Server is a black box

!SLIDE
# Actually...

!SLIDE incremental bullets
# it's an end-to-end test

!SLIDE incremental bullets
# the only way to verify..

* Client-side behavior
* Interaction with other server instances
* Redis, Rabbit, etc.

!SLIDE incremental bullets
# We'd also like to..

* Test controllers once!
* Fully & quickly
* Execute tests often

!SLIDE incremental bullets
# In summary..

* We can't replace the need for end-to-end tests
* But we can minimize errors
* Have confidence in @MVC code
* During end-to-end tests


!SLIDE incremental bullets
# Spring MVC Test

* Built on `spring-test`
* No Servlet container
* Drives Spring MVC infrastructure
* Both server & client-side test support <br> (i.e. `RestTemplate` code)
* _Inspired by `spring-ws-test`_

!SLIDE incremental bullets
# The Approach

* Re-use controller test fixtures
* I.e. mocked services & repositories
* Invoke `@Controller` classes through @MVC infrastructure

!SLIDE small
# Example

    @@@ java

    String contextLoc = "classpath:appContext.xml";
    String warDir = "src/main/webapp";

    MockMvc mockMvc =
      xmlConfigSetup(contextLoc)
        .configureWebAppRootDir(warDir, false)
        .build();

    mockMvc.perform(post("/persons"))
      .andExpect(response().status().isOk())
      .andExpect(response().forwardedUrl("/add.jsp"))
      .andExpect(model().size(1))
      .andExpect(model().hasAttributes("person"));

!SLIDE incremental bullets
# Under the Covers

* Mock `request/response` from `spring-test`
* `DispatcherServlet` replacement
* Multiple ways to initialize MVC infrastructure
* Save results for expectations

!SLIDE incremental bullets
# Ways To Initialize MVC Infrastructure

!SLIDE small
# From Existing XML Config

    @@@ java

    // XML config

    MockMvc mockMvc = 
        xmlConfigSetup("classpath:appContext.xml")
          .activateProfiles(..)
          .configureWebAppRootDir(warDir, false)
          .build();

!SLIDE small
# From Existing Java Config

    @@@ java

    // Java config

    MockMvc mockMvc = 
        annotationConfigSetup(WebConfig.class)
          .activateProfiles(..)
          .configureWebAppRootDir(warDir, false)
          .build();

!SLIDE small
# Manually 

    @@@ java

        MockMvc mockMvc =
            standaloneSetup(new PersonController())
              .setMessageConverters(..)
              .setValidator(..)
              .setConversionService(..)
              .addInterceptors(..)
              .setViewResolvers(..)
              .setLocaleResolver(..)
              .build();

!SLIDE incremental bullets
# About Manual Setup

* MVC components instantiated directly 
* Not looked up in Spring context
* Focused, readable tests
* But must verify MVC config separately

!SLIDE incremental bullets smaller
# `TestContext` Framework Example

    @@@ java

    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration(
		    locations={"/org/example/servlet-context.xml"})
    public class TestContextTests {

      @Autowired
      private WebApplicationContext wac;

      @Before
      public void setup() {
        MockMvc mockMvc = 
            MockMvcBuilders.webApplicationContextSetup(this.wac)
              .build();
      }

    }

!SLIDE incremental bullets
# `TestContext` Framework

* Caches loaded Spring configuration
* Potentially across all tests!

!SLIDE incremental bullets
# However..

* `WebApplicationContext` not supported yet
* <a href="http://jira.springframework.org/browse/SPR-5243">To be supported</a> in Spring 3.2

!SLIDE incremental bullets
# In the mean time..

* You can use a custom `ContextLoader`
* <a href="https://github.com/SpringSource/spring-test-mvc/blob/master/src/test/java/org/springframework/test/web/server/samples/context/TestContextTests.java">Example exists</a> in `spring-test-mvc`

!SLIDE incremental bullets
# Step 1

* Add static imports
* `MockMvcBuilders.*`
* `MockMvcRequestBuilders.*`
* `MockMvcResultActions.*`

!SLIDE incremental bullets
# Easy to remember..

* `MockMvc*`

!SLIDE incremental bullets
# Also in Eclipse..

* Add `MockMvc*` classes in _Preferences_
* _Java -> Editor -> Favorites_
* Helps content assist

!SLIDE smaller
# Step 2 
## Initialize MVC infrastructure

    @@@ java

    String contextLoc = "classpath:appContext.xml";
    String warDir = "src/main/webapp";

    MockMvc mockMvc = xmlConfigSetup("classpath:appContext.xml")
        .configureWebAppRootDir(warDir, false)
        .build()









    // ...

!SLIDE smaller
# Step 3
## Build Request

    @@@ java

    String contextLoc = "classpath:appContext.xml";
    String warDir = "src/main/webapp";

    MockMvc mockMvc = xmlConfigSetup("classpath:appContext.xml")
        .configureWebAppRootDir(warDir, false)
        .build()

    mockMvc.perform(get("/").accept(MediaType.APPLICATION_XML))







    // ...

!SLIDE smaller
# Step 4
## Add Expectations

    @@@ java

    String contextLoc = "classpath:appContext.xml";
    String warDir = "src/main/webapp";

    MockMvc mockMvc = xmlConfigSetup("classpath:appContext.xml")
        .configureWebAppRootDir(warDir, false)
        .build()

    mockMvc.perform(get("/").accept(MediaType.APPLICATION_XML))
          .andExpect(response().status().isOk())
          .andExpect(response().contentType(MediaType))
          .andExpect(response().content().xpath(String).exists())
          .andExpect(response().redirectedUrl(String))
          .andExpect(model().hasAttributes(String...));


    // ...


!SLIDE smaller
# Step 5
## Debug

    @@@ java

    String contextLoc = "classpath:appContext.xml";
    String warDir = "src/main/webapp";

    MockMvc mockMvc = xmlConfigSetup("classpath:appContext.xml")
        .configureWebAppRootDir(warDir, false)
        .build()

    mockMvc.perform(get("/").accept(MediaType.APPLICATION_XML))
          .andPrint(toConsole());






    // ...


!SLIDE code
# Demo

<a href="https://github.com/SpringSource/spring-test-mvc">https://github.com/SpringSource/spring-test-mvc</a>

__Sample Tests:__<br>
<a href="https://github.com/SpringSource/spring-test-mvc/tree/master/src/test/java/org/springframework/test/web/server/samples">org.springframework.test.web.server.samples</a>

!SLIDE small
# Client-Side Example

    @@@ java

    RestTemplate restTemplate = ... ;

    MockRestServiceServer.createServer(restTemplate)
        .expect(requestTo(String))
        .andExpect(method(HttpMethod))
        .andExpect(body(String))
        .andExpect(headerContains(String, String))
        .andRespond(withResponse(String, HttpHeaders));

!SLIDE incremental bullets small
# Project Availability

* <a href="github.com/SpringSource/spring-test-mvc">`github.com/SpringSource/spring-test-mvc`</a>
* Request for feedback!
* Send comments
* Pull requests

!SLIDE incremental bullets smaller
# Nightly Snapshot Available

    @@@ xml

    <repository>
        <id>org.springframework.maven.snapshot</id>
        <name>Spring Maven Snapshot Repository</name>
        <url>http://maven.springframework.org/snapshot</url>
        <releases>
            <enabled>false</enabled>
        </releases>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>

