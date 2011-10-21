
!SLIDE subsection bullets
# Agenda

* ..
* ..
* __Spring MVC Test Support__

!SLIDE small
# How To Test `@Controller`?

    @@@ java

    @Controller
    @SessionAttributes("formBean")
    public class FormController {
    
	    @ModelAttribute("formBean")
	    public FormBean createFormBean() {

		    // ...
	    }

    	@RequestMapping(...)
	    public String processSubmit(
                        @Valid FormBean formBean, 
                        BindingResult result) {

            // ...
        }
    }

!SLIDE incremental
# Unit Test

* Instantiate controller manually
* Prepare inputs, invoke methods
* Simple but not fully tested
* MVC config, annotations, etc.

!SLIDE incremental
# End-to-End Tests

* Selenium, JWebUnit, etc.
* Everything is tested but more involved
* Harder to maintain
* _Wait, didn't we unit test controllers?_

.notes Requires running container, takes more time and resources, not likely to be run frequently.

!SLIDE incremental
# Ideally We'd Like To...

* Test controllers ones, fully
* Include MVC config, annotations, etc.
* Remain lightweight and fast
* Controller Unit Test++

!SLIDE incremental 
# Spring MVC Test Support

* Small framework built on `spring-test`
* No servlet container
* Drives Spring MVC infrastructure
* Both server & client-side test support <br> (i.e. `RestTemplate` code)
* Inspired by `spring-ws-test`

!SLIDE incremental
# Unit vs. Integration Style

* A bit of both
* Aids existing `@Controller` unit tests
* Does not replace end-to-end tests
* Ensures Spring MVC stuff works
* Without the overhead

.notes For example, the vFabric management team has a JSON-based REST API; unit tests verify JSON rendering using @Controllers; integration tests involve multiple tc Server and other instances, which need to run regardless.

!SLIDE small
# Server-Side Test Example

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

!SLIDE incremental
# Static Imports
## (_Ctrl+Shift+T_ __`"MockMvc*"`__)

* `MockMvcBuilders.*`
  * Spring MVC setup
* `MockMvcRequestBuilders.*`
  * request building
* `MockMvcResultActions.*`
  * expectation setting

!SLIDE small
# Spring MVC Setup
## (from existing configuration)

    @@@ java

    // XML config

    MockMvc mockMvc = 
        xmlConfigSetup("classpath:appContext.xml")
          .activateProfiles(..)
          .configureWebAppRootDir(warDir, false)
          .build();

    // Java config

    MockMvc mockMvc = 
        annotationConfigSetup(WebConfig.class)
          .activateProfiles(..)
          .configureWebAppRootDir(warDir, false)
          .build();

.notes The ApplicationContext is scanned for MVC infrastructure components like the DispatcherServlet does.

!SLIDE small
# Spring MVC Setup
## (standalone)

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

.notes MVC infrastructure components are directly instantiated without scanning an ApplicationContext.

!SLIDE incremental
# Under the Covers

* Mock `request/response` from `spring-test`
* `DispatcherServlet` replacement
* Drives Spring MVC infrastructure
* Exposes results for expectation setting

!SLIDE smaller
# Expectations

    @@@ java

    MockMvc mockMvc = 
      standaloneSetup(new PersonController()).build();

    mockMvc.perform(get("/"))
      .andExpect(response().status().isOk())
      .andExpect(response().contentType(MediaType))
      .andExpect(response().content().isEqualTo(String))
      .andExpect(response().content().isEqualToXml(String))
      .andExpect(response().content().xpath(String).exists())
      .andExpect(response().content().jsonPath(String).exists())
      .andExpect(response().forwardedUrl(String))
      .andExpect(response().redirectedUrl(String))
      .andExpect(model().size(int))
      .andExpect(model().hasAttributes(String...))
      .andPrint(toConsole());

    // Many more available...

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

!SLIDE incremental small
# Project Availability

* Source on github
  * `github.com/SpringSource/spring-test-mvc`
* Nightly snapshot published
  * `http://maven.springframework.org/milestone`
* Request for feedback!

!SLIDE incremental
# Roadmap

* Equalize client & server-side options (xpath, jsonpath, etc.)
* More request building options
* _&lt;your feedback&gt;_...
* The more, the better

!SLIDE incremental
# Timeline

* Release candidate by year end
* Possible inclusion in Spring 3.2



