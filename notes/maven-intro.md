# Introduction to Maven

Apache Maven is a powerful project management tool primarily used for Java projects. Maven addresses two aspects of building software: first, it describes how software is built, and second, it describes its dependencies. It simplifies the build process, provides a uniform build system, manages dependencies, and enables easy project configuration.

## Advantages of Maven

- **Standardized Build Process:** Maven uses a standardized build process, making it easier for developers to understand and work on projects.
- **Dependency Management:** Automatically handles the inclusion of necessary libraries and manages transitive dependencies.
- **Project Consistency:** Ensures consistency across projects with a common project structure and lifecycle.
- **Extensibility:** Provides a rich set of plugins that can be used to customize and extend the build process.
- **Integration with IDEs:** Supports integration with various Integrated Development Environments (IDEs) like Eclipse, IntelliJ IDEA, and NetBeans.
- **Ease of Use:** Simplifies the build process with a straightforward configuration file (POM).

- Maven uses POM (Project Object Model) to achieve this.
- When "Maven" command is given, it looks for "pom.xml" in the current directory & gets needed configuaration

# Difference between ANT and Maven

- In Ant, we have to tell "What to do" and "How to do". Whereas in Maven, we only tell "what to do".

- Ant provides flexibility and control over the build process, it requires more manual configuration and management of dependencies. Maven, on the other hand, simplifies the build process with a standardized lifecycle and built-in dependency management, making it easier to manage and maintain large projects.


# Maven Project Structure

A typical Maven project follows a standardized directory layout, which helps in maintaining consistency across projects. Below is the structure of a basic Maven project:

![Maven File Structure Diagram](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/MavenFileStructure.png)


# POM file understanding

```
<?xml version="1.0" encoding="UTF-8"?>

<!-- It specifies the XML schema, also make sure that our XML adheres to correct structure and version defined by maven -->

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

    <!-- Used to define the Parent project. Current project inherit the configurations from the specified parent project. Which in turn inherited from Super Pom-->
    <!-- Each POM has one Parent -->
    <!-- If this <parent> field is not specified, maven by-default inherit the configurations from "Super Pom". This is the link of maven Super POM: https://maven.apache.org/ref/3.0.4/maven-model-builder/super-pom.html -->
	
    <parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>     <!-- This is the current pom.xml file's parent  -->
		<version>3.3.1</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

    <!-- Unique identifier of your project -->

	<groupId>com.example</groupId>
	<artifactId>learningspringboot</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>springboot application</name>
	<description>project for learning spring boot</description>
	<url/>
	<licenses>
		<license/>
	</licenses>
	<developers>
		<developer/>
	</developers>
	<scm>
		<connection/>
		<developerConnection/>
		<tag/>
		<url/>
	</scm>
	
    
    <properties>
		<java.version>17</java.version>     <!-- Define Key-Value pair for configuration. Can be referenced throught out the pom.xml file using ${java.version} -->
	</properties>

    <!-- This is where we declare the dependencies that our project relies on -->

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>

<!--
    We can have <repositories>, where maven look for project dependencies and download the artifacts (jars). This is generally by default maven adds. May be you might find in super-pom.xml .

    <repositories>
        <repository>
            <id>central</id>
            <name>Central Repository</name>
            <url>https://repo.maven.apache.org/maven2</url>
        </repository>
    </repositories>
-->
```

# Maven Build Lifecycle Phases

Maven's build lifecycle is a sequence of phases that define the steps in building and distributing a project. Each build lifecycle is composed of a series of build phases, where each phase represents a specific stage in the lifecycle. The default lifecycle is the most commonly used, and it includes several important phases. Below is a detailed explanation of the default lifecycle phases:

## Default Lifecycle Phases

1. **validate**: Validates the project to ensure it is correct and all necessary information is available. This phase checks the project's structure and configuration.

2. **compile**: Compiles the source code of the project. In this phase, Maven uses the `maven-compiler-plugin` to compile the project's source code into bytecode that the JVM can execute.

3. **test**: Tests the compiled source code using a suitable unit testing framework (e.g., JUnit). These tests should not require the code to be packaged or deployed. This phase ensures that the code is working as expected before packaging.

4. **package**: Takes the compiled code and packages it into its distributable format, such as a JAR, WAR, or EAR file. This phase is responsible for creating the final artifact that can be distributed.

5. **verify**: Runs any checks on the results of integration tests to ensure that quality criteria are met. This phase may include running additional testing or code analysis tools.

6. **install**: Installs the package into the local repository, for use as a dependency in other projects locally. This phase allows other projects on the same machine to use the built artifact as a dependency.

7. **deploy**: Copies the final package to a remote repository for sharing with other developers and projects. This phase is typically executed in a build environment and involves deploying the artifact to a remote Maven repository, such as Nexus or Artifactory.

## Sequential Execution of Phases

These lifecycle phases are executed sequentially to complete the default lifecycle i.e If you want to run "Package" phase all its previous phases will get executed first.

And if you want to run specific goal of a particular phase, then all the goals of previous phases + current phase goals before the one you defined will get run.

When the default lifecycle is invoked, Maven will execute the phases in the following order:

1. **validate**
2. **compile**
3. **test**
4. **package**
5. **verify**
6. **install**
7. **deploy**

This means that when the default lifecycle is used, Maven will first validate the project, then compile the sources, run tests against the compiled code, package the binaries (e.g., JAR file), run integration tests and verify the results, install the verified package to the local repository, and finally deploy the package to a remote repository.

**Build element in pom.xml, helps you add a new task/goal inside one of the phase.**

Maven already has validate phase defined and its goal, but if we want to add more goals or task, then we can use <build> element. And add the goal to specific phase.

<u>Validate:</u> mvn validate

```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>2.7.3</version>
            <executions>
                <execution>
                    <id>validate-checkstyle</id>
                    <phase>validate</phase>
                    <goals>
                        <goal>checkstyle</goal>
                    </goals>
            </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

For a complete list of all lifecycle phases and more detailed information, you can refer to the [Maven Lifecycle Reference](https://maven.apache.org/ref/current/maven-core/lifecycles.html).
