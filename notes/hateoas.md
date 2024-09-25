# Spring Boot HATEOAS Restful API

## Introduction
HATEOAS (Hypermedia as the Engine of Application State) is a constraint of the REST application architecture. It is a way to make RESTful services more self-descriptive and discoverable by including hypermedia links in the responses. These links provide clients with information about what actions they can perform next, making the API more intuitive and easier to navigate.

![hateoas-example](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/hateoas-example.png)

### Explanation
Lets say we have reached to this step using a `POST` API, `/addUser` where we are able to add a user but currently its `Unverified`. The next step after adding a user can be 
1. Make a call to fetch user details (GET: /getuser).
2. Make a call to delete user details (DELETE: /removeuser).
3. Make a call to verify user,

For `VERIFY` user, we can have different ways to verify
1. `SMS-VERIFY`
2. `EMAIL-VERIFY`

![hateoas-response](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/hateoas-response.png)

Before understanding HOW to do this, lets understand WHEN to use and WHY to use HATEOAS link?

## Why Use HATEOAS (Advantages and Disadvantages)

### Advantages
1. **Discoverability**: Clients can discover available actions through hypermedia links without needing additional documentation.
2. **Decoupling**: Reduces the coupling between client and server, as clients can dynamically navigate the API based on the links provided.
3. **Self-Documentation**: The API becomes self-documenting, as the links describe the possible actions and transitions.
4. **Flexibility**: Easier to evolve the API over time without breaking existing clients.

To achieve above, server provides the next set of APIs(actions) in the Response itself, which client can take. So that client have less business logic around APIs(which API to invoke, when to invoke, how to invoke etc...)

But, Adding all next set of ACTIONS can make our API Response Bloat up and has several disadvantages:

### Disadvantages
1. **Complexity**: Adds complexity to the API design and implementation.
2. **Overhead**: Increases the size of the responses due to additional hypermedia links.
3. **Client Support**: Requires clients to understand and properly handle hypermedia links.
4. **Latency Impact**

![hateoas-example](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/hateoas-example.png)

Here in the above image, the `tight coupling` lies during VERIFY process. For both `SMS-VERIFY FINISHED` and `EMAIL-VERIFY FINISHED`, we need to have `SMS-VERIFY STARTED` and `EMAIL-VERIFY STARTED` respectively. Client need some info, before it can decide which Verify API to invoke

### Without HATEOAS

![response-without-hateoas](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/response-without-hateoas.png)

Client need to put similar business logic as below

![client-logic-without-hateoas](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/client-logic-without-hateoas.png)

### With HATEOAS

![response-with-hateoas](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/response-with-hateoas.png)

Now, we have achieved LOOSE COUPLING and Client code looks like this.

![client-logic-with-hateoas](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/client-logic-with-hateoas.png)

## How to Implement It


First, add the necessary dependencies to your `pom.xml` file.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-hateoas</artifactId>
    <version>2.6.4</version>
</dependency>
```

![hateoas-code](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/hateoas-code.png)

![other-way-to-create-link](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/other-way-to-create-link.png)

![hateoas-output](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/hateoas-output.png)


## Conclusion
Implementing HATEOAS in a Spring Boot RESTful API enhances the discoverability and flexibility of the API. By including hypermedia links in the responses, clients can dynamically navigate the API and understand the available actions without needing additional documentation. While it adds some complexity and overhead, the benefits of a more intuitive and decoupled API often outweigh the drawbacks.
