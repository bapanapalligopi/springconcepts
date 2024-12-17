project setup
![image](https://github.com/user-attachments/assets/112ea070-2ea6-417f-be66-edd0b039990c)

extract the project and ope in sts ,eclipse or even in intellij idea

![image](https://github.com/user-attachments/assets/68db7473-1d8d-4f9c-9e7a-a3634f9c2ba2)

run the zookeeper in your machine with following command
C:\kafka\bin\windows>zookeeper-server-start.bat C://kafka//config//zookeeper.properties
and you will get the image like 
![image](https://github.com/user-attachments/assets/01263fb8-af04-49ac-a0fc-415fc2fd6633)

start the kafka broker
C:\kafka\bin\windows>kafka-server-start.bat C://kafka//config//server.properties
![image](https://github.com/user-attachments/assets/f033e58b-4f93-4f22-8ae7-330b0d783268)
kafka architecture
![image](https://github.com/user-attachments/assets/20e3053d-0861-4c06-83c1-43e5893cdd9f)

creation of topic in kafka cluster
package com.kafka_project_example.kafka_project.config;

import org.apache.kafka.clients.admin.NewTopic;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.config.TopicBuilder;
import org.springframework.stereotype.Component;

@Component
@Configuration
public class KafkaConfig {

	@Bean
	public NewTopic topic() {
		
		return TopicBuilder
				.name("kAFKA_PROJECT_TOPIC")
				.build();
		
	}
}
create a kafka producer

package com.kafka_project_example.kafka_project.services;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Service;

@Service
public class KafkaProducer {

	//inject kafka_template for auto config
	@Autowired
	KafkaTemplate<String, Object> kafkaTemplate;
	
	private static final Logger LOGGER =LoggerFactory.getLogger(KafkaProducer.class);
	//sending messages to topic by producers
	public String sendMessage(String message) {
		//get kafka template
		kafkaTemplate.send("kAFKA_PROJECT_TOPIC", message);
		LOGGER.info(String.format("Message sent {}",message));
		return "message send successfully";
	}
	
}
creating a rest api to send message to a kafka topic by producer
