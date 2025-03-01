```java
 @Bean
    @Qualifier("aggregateSensorStep")
    public Step aggregateSensorStep(JobRepository jobRepository,PlatformTransactionManager platformTransactionManager) {
    	
    	logger.info("Defining aggregateSensorStep...");
    	return new StepBuilderFactory(jobRepository, platformTransactionManager)
    			.get("aggregateSensorStep")
    			.<DailySensorData, DailyAggregatedSensorData>chunk(1)
    			.reader(readerProccessorWriterConfiguration.textFileReader())
    			.processor(itemProcessorForTextFile)
    			.writer(itemWriterForXmlFIle2.getItemWriterBuilder())
    			.build();
    }
```
