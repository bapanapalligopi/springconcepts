![image](https://github.com/user-attachments/assets/9be38e51-fbce-4931-a8df-ebd5ba31c216)
![image](https://github.com/user-attachments/assets/7113d075-8d81-43f8-aad7-fd8cbe97ab5d)
![image](https://github.com/user-attachments/assets/4d2fe961-46d7-4755-bb3d-8dac60dd685e)
![image](https://github.com/user-attachments/assets/c5d3cc18-c136-4f9d-a04b-13f5818c841c)
![image](https://github.com/user-attachments/assets/983a4d4f-defe-4843-ade9-fe0bcdef3297)
![image](https://github.com/user-attachments/assets/28051065-71b7-43b4-8853-162fbe8cba8a)
make project has multi module project

so addd <packaging>pom</packaging> in pom.xml

![image](https://github.com/user-attachments/assets/524073d0-d74b-4709-97ca-25b418667abf)

![image](https://github.com/user-attachments/assets/2aeb2be9-68da-4c08-8a6f-675b1175de59)

![image](https://github.com/user-attachments/assets/51c54644-c46b-4e05-ad01-4ddc6e6dfed5)
updates pom.xml 
![image](https://github.com/user-attachments/assets/73ee3a66-01f0-49e2-9e09-f16fc2e16e85)
create a main application class in child module /kafka-producer-wikimedia

![image](https://github.com/user-attachments/assets/85d84ca0-d5a4-467c-b56b-a0c1b1497a0c)

package kafkaproducerwikimedia;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class KafkaProducerWikimediaApplication {
	public static void main(String[] args) {
		
		SpringApplication.run(KafkaProducerWikimediaApplication.class);
	}
}

do maven clean to check any build issue

![image](https://github.com/user-attachments/assets/f56c62ea-6a19-4517-92ac-2e90fa5425a9)

![image](https://github.com/user-attachments/assets/4fe8ab0a-8e0c-4f37-af9b-2f20dd60f520)
add packaging as jar in child-1 project 

![image](https://github.com/user-attachments/assets/c0101e7c-a78a-4acf-9c6a-914e792dac02)
create application.properties file

![image](https://github.com/user-attachments/assets/3876e176-bf98-4645-a890-9f14391f2252)
https://github.com/Gopi17-Developer/wikimedia_kafka_project
