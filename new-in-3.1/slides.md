!SLIDE subsection transition=scrollDown
# New in Spring 3.1


!SLIDE bullets center
* `SmartContextLoader`

!SLIDE bullets center
* `AnnotationConfigContextLoader`

!SLIDE bullets center
* testing with `@Configuration` classes

!SLIDE bullets center
* environment profiles

!SLIDE bullets center
* `DelegatingSmartContextLoader`

!SLIDE bullets center
* updated context cache key generation


!SLIDE incremental small
# `SmartContextLoader` SPI

* Strategy for loading application contexts
  * from resource locations _or_ `@Configuration` classes
* Supports environment profiles
* Supersedes `ContextLoader`
* Out of the box:
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


!SLIDE incremental
# Example: @Configuration

* ...


!SLIDE incremental
# @Configuration + XML

* ...


!SLIDE subsection
# Testing with<br /> Environment Profiles


!SLIDE incremental
# @ActiveProfiles

* ...


!SLIDE subsection
# Context Caching


!SLIDE bullets center
* ...

