To configure a Kafka producer and consumer for JSON serialization/deserialization in Spring Kafka, you need to ensure both the producer and consumer are set up to handle JSON messages properly. Here's a step-by-step guide on configuring your `application.properties`, setting up the producer, and configuring the consumer.

### 1. **Update `application.properties`**

Make sure to update the `application.properties` file to configure the Kafka producer and consumer to handle JSON serialization and deserialization.

```properties
# Kafka Consumer Configuration
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer
spring.kafka.consumer.properties.spring.json.trusted.packages=*
spring.kafka.consumer.group-id=kafka_project

# Kafka Producer Configuration
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer
```

In this configuration:
- The `JsonDeserializer` is used for the consumer to deserialize the `User` object.
- The `JsonSerializer` is used for the producer to serialize the `User` object before sending it to Kafka.

### 2. **Create the Kafka Producer**

Here is your existing Kafka producer code, which already handles sending `User` objects to Kafka in JSON format.

```java
package com.kafka_project_example.kafka_project.services;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.support.KafkaHeaders;
import org.springframework.messaging.Message;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.stereotype.Service;

import com.kafka_project_example.kafka_project.models.User;

@Service
public class KafkaProducerJson {

    private static final Logger LOGGER = LoggerFactory.getLogger(KafkaProducerJson.class);

    @Autowired
    private KafkaTemplate<String, User> kafkaTemplate;

    public void sendMessage(User data) {
        LOGGER.info(String.format("Message sent -> %s", data.toString()));
        Message<User> message = MessageBuilder
                .withPayload(data)
                .setHeader(KafkaHeaders.TOPIC, "kAFKA_PROJECT_TOPIC")
                .build();
        kafkaTemplate.send(message);
    }
}
```

This service sends a `User` object to the Kafka topic `"kAFKA_PROJECT_TOPIC"`. The object will be serialized to JSON because of the `JsonSerializer` specified in `application.properties`.

### 3. **Create the REST Controller**

Here’s the REST controller to send `User` objects via the Kafka producer.

```java
package com.kafka_project_example.kafka_project.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import com.kafka_project_example.kafka_project.models.User;
import com.kafka_project_example.kafka_project.services.KafkaProducerJson;

@RestController
@RequestMapping("/api/v1/json-message")
public class JsonMessageController {

    @Autowired
    private KafkaProducerJson kafkaProducerJson;

    @GetMapping("/send")
    public ResponseEntity<String> publish(@RequestBody User user) {
        kafkaProducerJson.sendMessage(user);
        return ResponseEntity.ok("Success json");
    }
}
```

This REST controller exposes an endpoint `GET /api/v1/json-message/send`, where you can send a `User` object in the request body, which is then passed to the Kafka producer to send the message to the Kafka topic.

### 4. **Create the Kafka Consumer**

Finally, configure the Kafka consumer to handle the incoming `User` object. This consumer will automatically deserialize the JSON payload into a `User` object.

```java
package com.kafka_project_example.kafka_project.services;

import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Service;

import com.kafka_project_example.kafka_project.models.User;

@Service
public class KafkaConsumerJson {

    @KafkaListener(topics = "kAFKA_PROJECT_TOPIC", groupId = "kafka_project")
    public void kafkaConsumer(User user) {
        // Process the User object received from Kafka
        System.out.println("Received User: " + user);
    }
}
```

This consumer listens to the `kAFKA_PROJECT_TOPIC` topic and automatically converts the JSON message into a `User` object using the `JsonDeserializer` configured in the `application.properties`.

### 5. **Ensure Dependencies in `pom.xml`**

Ensure that the following dependencies are present in your `pom.xml` to work with Kafka and JSON serialization/deserialization.

```xml
<dependencies>
    <!-- Spring Boot Starter for Kafka -->
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka</artifactId>
    </dependency>
    
    <!-- Jackson JSON processor -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
    </dependency>
    
    <!-- Spring Boot Starter Web (for REST API) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <!-- Spring Boot Starter for Kafka Serialization/Deserialization -->
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka</artifactId>
    </dependency>
</dependencies>
```

### Summary:
1. **Producer Configuration**: The producer sends the `User` object as JSON using `JsonSerializer`.
2. **Consumer Configuration**: The consumer receives the `User` object as JSON using `JsonDeserializer`.
3. **REST Controller**: Allows sending `User` objects via HTTP requests.
4. **Kafka Consumer**: Listens to the Kafka topic and automatically deserializes the incoming JSON message into a `User` object.

With this setup, you can produce and consume JSON messages for the `User` class using Spring Kafka.
