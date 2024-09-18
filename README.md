# Spring Boot Daily Learnings

Welcome to my Spring Boot Daily Learnings repository! This repository is dedicated to tracking and documenting my daily progress, experiments, and experiences as I learn and work with Spring Boot.

## About

Spring Boot is a powerful framework that simplifies the development of Java-based applications. This repository serves as a journal of my journey through various concepts, features, and best practices in Spring Boot.

## File Structure

```
spring-boot-daily-learnings/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── example/
│   │   │           └── learningspringboot/
│   │   │               ├── controller/
│   │   │               ├── model/
│   │   │               ├── repository/
│   │   │               └── service/
│   │   └── resources/
│   │       ├── static/
│   │       └── templates/
│   └── test/
│       └── java/
│           └── com/
│               └── example/
│                   └── learningspringboot/
├── .gitignore
├── mvnw
├── mvnw.cmd
├── pom.xml
└── README.md
└── notes/
```

## Notes

Here are links to my notes taken during my learning journey with Spring Boot:

- [Spring Boot Learning RoadMap](notes/roadmap.md)
- [Spring Boot Introduction](notes/springboot-intro.md)
- [Spring Boot setup and architecture](notes/layered-architecture.md)
- [Introduction to Maven](notes/maven-intro.md)
- [Annotations Part1](notes/annotations(controller-layer).md)
- [Bean and its Lifecycle](notes/bean-intro.md)
- [Dependency Injection](notes/dependence-injection.md)
- [Bean Scopes](notes/bean-scope.md)
- [Dynamically Initialized Beans](notes/dynamically-initialized-beans.md)
- [@ConditionalOnProperty Annotation](notes/ConditionalOnProperty.md)
- [@Profile Annotation](notes/profile-annotation.md)
- [Spring Boot AOP (Aspect Oriented Programming)](notes/aspect-oriented-programming.md)
- [Spring Boot Transactional Part1](notes/transactional-annotation-1.md)
- [Spring Boot Transactional Part2](notes/transactional-annotation-2.md)
- [Spring Boot Transactional Part3](notes/transactional-annotation-3.md)
- [Spring Boot Async Annotation Part1](notes/async-annotation-1.md)
- [Spring Boot Async Annotation Part2](notes/async-annotation-2.md)

## Installation

To install and run the Spring Boot application, follow these steps:

1. Clone the repository:

    ```
    git clone https://github.com/DharaniDJ/spring-boot-daily-learnings.git
    ```

2. Navigate to the project directory:

    ```
    cd spring-boot-daily-learnings
    ```

3. Build the project:

    ```
    ./mvnw clean install
    ```

4. Run the application:

    ```
    ./mvnw spring-boot:run
    ```

5. Access the application in your browser:

    ```
    http://localhost:8080
    ```

Happy learning!
