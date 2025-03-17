

# **Microservices with Eureka Service Discovery**

This project demonstrates how to integrate **Spring Cloud Eureka** for **service discovery** between multiple Spring Boot applications. The two client applications—**UserAuthenticationService** and **ProductCatalog**—consume services registered with a central **Eureka Server**.

### **Architecture Overview**

- **Eureka Server**: The service registry that holds information about all the available services in the system.
- **UserAuthenticationService**: A Spring Boot microservice that registers itself with Eureka and consumes other services from the registry.
- **ProductCatalog**: Another Spring Boot microservice that also registers with Eureka and consumes services from the registry.

### **Project Structure**

- **Eureka Server**: Acts as the service registry for all other services in the ecosystem.
- **UserAuthenticationService**: A service responsible for handling user authentication.
- **ProductCatalog**: A service that deals with product-related data and operations.

### **Eureka Server Setup**

1. **Add Dependencies** in `pom.xml`:
   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
   </dependency>
   ```

2. **Enable Eureka Server** in your Spring Boot application:
  

3. **Configure Eureka Server** in `application.properties`:
   ```
   server.port: 8761  # Eureka Server will run on this port

   eureka.client.register-with-eureka: false  # Eureka Server does not register with itself
   eureka.client.fetch-registry: false  # No need for Eureka Server to fetch the registry
   ```

4. **Start Eureka Server**: Run the `EurekaServerApplication` class. The Eureka server will be available at `http://localhost:8761`.

---

### **UserAuthenticationService Setup**

1. **Add Dependencies** in `pom.xml`:
   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
   </dependency>
   ```

3. **Configure Eureka Client** in `application.properties`:
   ```
   eureka.client.service-url.defaultZone: http://localhost:8761/eureka/  # URL of the Eureka server
   ```

4. **Start UserAuthenticationService**: Run the `UserAuthenticationServiceApplication` class. The service will register with Eureka and be discoverable by other services.

---

### **ProductCatalog Setup**

1. **Add Dependencies** in `pom.xml`:
   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
   </dependency>
   ```

3. **Configure Eureka Client** in `application.properties`:
   ```
      eureka.client.service-url.defaultZone: http://localhost:8761/eureka/  # URL of the Eureka server
   ```

4. **Start ProductCatalog**: Run the `ProductCatalogApplication` class. The service will register itself with Eureka and be discoverable by other services.

---

### **Service Discovery**

Once the services are up and running and have registered with Eureka, the services can discover each other dynamically using Eureka's service registry.

For example, the **ProductCatalog** service can discover the **UserAuthenticationService** dynamically and call it using a `RestTemplate`.

In `ProductCatalogService.java`:

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
public class ProductCatalogService {

    @Autowired
    private RestTemplate restTemplate;

    @GetMapping("/getUserDetails")
    public String getUserDetails() {
        // Use Eureka to resolve service name and make the request
        String userServiceUrl = "http://UserAuthenticationService/method_name";
        return restTemplate.getForObject(userServiceUrl, String.class);
    }
}
```

This `RestTemplate` request will resolve `UserAuthenticationService` using Eureka and communicate with the correct instance.

### **Enabling Load Balancing with Spring Cloud**

To enable **load balancing** for requests made via **Eureka**, you can annotate the `RestTemplate` with `@LoadBalanced`:

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.web.client.RestTemplate;

@Configuration
public class AppConfig {

    @Bean
    @LoadBalanced  // Enable client-side load balancing
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

This ensures that requests to `http://user-authentication-service` are load balanced across all available instances of that service.

### **Running the Application**

1. **Start Eureka Server**: Run the `EurekaServerApplication` class.
2. **Start UserAuthenticationService**: Run the `UserAuthenticationServiceApplication` class.
3. **Start ProductCatalog**: Run the `ProductCatalogApplication` class.
4. Open the Eureka dashboard: `http://localhost:8761`.

---

### **Summary**

- **Eureka Server** acts as the central registry for all services.
- **UserAuthenticationService** and **ProductCatalog** register with Eureka and can dynamically discover each other.
- Services can communicate without hardcoding IP addresses, and load balancing is enabled for client-side calls.
- Eureka provides health checks, self-preservation mode, and fault tolerance, ensuring the system remains resilient.

---
