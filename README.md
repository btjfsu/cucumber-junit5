#Cucumber JUnit Platform Engine

How to Guide to Use JUnit Platform to execute Cucumber scenarios.

## Overview
Cucumber is a testing framework that supports Behavior Driven Development (BDD), allowing users to define 
application operations in plain text. It works based on the Gherkin Domain Specific Language (DSL). This a simple, yet powerful syntax of Gherkin lets developers and testers 
write complex tests while keeping each test comprehensible to even non-technical users.

JUnit is one of the most popular unit-testing frameworks in the Java ecosystem. The JUnit 5 version contains a number of innovations, 
with the goal of supporting new features in Java 8 and above, as well as enabling many styles of testing. 

To ensure that developers are able to take advantage of the latest and great functionality offered by Cucumber JVM, This documentation was created to 
provide a comprehensive approach to getting started with the Cucumber JUnit Platform Engine and takign advantage of these features and functionality.

## Dependencies

```xml

<project>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>io.cucumber</groupId>
                <artifactId>cucumber-bom</artifactId>
                <version>7.0.0</version>
                <scope>import</scope>
                <type>pom</type>
            </dependency>
            <dependency>
                <groupId>org.junit</groupId>
                <artifactId>junit-bom</artifactId>
                <version>5.8.0</version>
                <scope>import</scope>
                <type>pom</type>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>io.cucumber</groupId>
            <artifactId>cucumber-junit-platform-engine</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>io.cucumber</groupId>
            <artifactId>cucumber-spring</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>io.cucumber</groupId>
            <artifactId>cucumber-java</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.platform</groupId>
            <artifactId>junit-platform-launcher</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.platform</groupId>
            <artifactId>junit-platform-suite</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

## Suites with different configurations

The JUnit Platform Suite Engine can be used to run Cucumber multiple times with different configurations. This is the recommended way to run Cucumber within
your project:

```java
package com.example;

import org.junit.platform.suite.api.ConfigurationParameter;
import org.junit.platform.suite.api.IncludeEngines;
import org.junit.platform.suite.api.SelectClasspathResource;
import org.junit.platform.suite.api.Suite;

import static io.cucumber.junit.platform.engine.Constants.GLUE_PROPERTY_NAME;

@Suite
@IncludeEngines("cucumber")
@SelectClasspathResource("com/example")
@ConfigurationParameter(key = GLUE_PROPERTY_NAME, value = "com.example")
public class RunCucumberTest {
}
```

## How to properly set up Cucumber Test Engine with JUnit 5
In order to properly set up Cucumber Test Engine, two classes are needed:
1) Runner class: Similar to the class above, this class focuses on the set-up of Cucumber with configuration, including the class path resources (e.g. feature file(s)), and the glue (package(s) of configuration classes and/or step definitions)
2) Context Configuration class: This class is used to set up the dependencies and initializers needed to properly start up the application to execute Cucumber scenarios.
> Note: The Context Configuration class must include the `@CucumberContextConfiguration` annotation

> Note: The Context Configuration class must be located within the package (or one of the packages) defined in the cucumber.glue

```java
package com.example;

import io.cucumber.java.AfterAll;
import io.cucumber.java.BeforeAll;
import io.cucumber.spring.CucumberContextConfiguration;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.ActiveProfiles;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles(profiles = {"acceptance-test"})
@CucumberContextConfiguration
public class CucumberEnvironment {
    
    // write @ParameterType and @DataTableTypes here
    // write @BeforeAll, @Before, @After, @AfterAll here 
    // (io.cucumber.java annotations, not JUnit Jupiter annotations!)
}
```

## Important Note about Cucumber Test Engine with JUnit 5
The JUnit Platform Suite Engine is separate from the JUnit Jupiter Platform Engine, so functionality typically used within JUnit 5 tests will NOT work\
when running Cucumber tests. This includes, but is not limited to:
* JUnit Jupiter Annotations
  * `@BeforeAll`
  * `@BeforeEach`
  * `@AfterAll`
  * `@AfterEach`

* JUnit Jupiter Extensions
  * `@ExtendWith`
  * `@RegisterExtension`

## Best Practices for including Dependencies for running Cucumber
Due to the lack of support for JUnit 5 Extensions, components needed for testing (e.g Embedded Kafka, Embedded Cassandra, etc.) would need to be started under a different test-based context.
Below listed the ideal ways to inject these dependencies before executing the scenarios in Cucumber:

### Spring Test Context
The Spring TestContext Framework (located in the `org.springframework.test.context` package) provides generic, annotation-driven unit 
and integration testing support that is agnostic of the testing framework in use. 
The TestContext framework also places a great deal of importance on convention over configuration, with reasonable defaults that you can 
override through annotation-based configuration.

#### Embedded Kafka
You can run Embedded Kafka by adding the `@EmbeddedKafka`. This uses the Spring Test Context to start up, please refer to the following documentation for more information:

### Cucumber Context
As of Cucumber Java 7.0.0, the `@BeforeAll` and `@AfterAll` annotations have been added to facilitate consistent set-up and tear down of components in a clear and consistent way after running each feature.
```java
package com.example;

import io.cucumber.java.AfterAll;
import io.cucumber.java.BeforeAll;
import io.cucumber.spring.CucumberContextConfiguration;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.ActiveProfiles;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles(profiles = {"acceptance-test"})
@CucumberContextConfiguration
public class CucumberEnvironment {
    
    // write @ParameterType and @DataTableTypes here
    // write @BeforeAll, @Before, @After, @AfterAll here 
    // (io.cucumber.java annotations, not JUnit Jupiter annotations!)
  
  @BeforeAll
  public static void startEmUp() {
      // start something here. It will execute before all other contexts (including Spring!)
  }
  
  @AfterAll
  public static void tearEmDown() {
    // close, stop, or end something here. It will execute after all other contexts (including Spring!)
  }
}
```




