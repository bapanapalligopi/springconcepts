a mechanism used in microservices to avoid cascading failures and isolate the failures from one service to another servoce

closed State-> system healthy
Open State-> based on failure rate and thresh hold it can be closed to open , means systen facing failures
Half Open-> certian number of calls allowed

<!-- Source: https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-circuitbreaker-resilience4j -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
    <version>5.0.2</version>
    <scope>compile</scope>
</dependency>

configure circut breaker in appplication
1. add the dependencies
2. add the actitator
3. add the properties 
