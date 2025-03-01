```java
package com.practice.sensordata;

import javax.batch.runtime.JobExecution;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.springframework.batch.core.JobParameters;
import org.springframework.batch.core.JobParametersBuilder;
import org.springframework.batch.core.configuration.annotation.EnableBatchProcessing;
import org.springframework.batch.core.launch.JobLauncher;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@EnableBatchProcessing
public class SensorDataSpringBatchApplication implements CommandLineRunner{
	
	private final Logger logger= LogManager.getLogger(SensorDataSpringBatchApplication.class);
	@Autowired
	JobLauncher jobLauncher;
	@Autowired
	TemperatureSensorRootConfiguration temperatureSensorRootConfiguration;
	public static void main(String[] args) {
		SpringApplication.run(SensorDataSpringBatchApplication.class, args);
	}
 
	@Override
	public void run(String... args) throws Exception {
		// TODO Auto-generated method stub
		logger.info("Starting Batch Job...");
		JobParameters jobParameters = new JobParametersBuilder().addLong("time1",System.currentTimeMillis()).toJobParameters();
		JobExecution  jobExecution= (JobExecution) jobLauncher.run(temperatureSensorRootConfiguration.aggregateSensorJob(), jobParameters);
		
		logger.info("Batch Job Status: " + jobExecution.getExitStatus());

	}
}
```
