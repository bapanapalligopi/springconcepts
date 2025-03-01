```java
  @Bean
    public Job aggregateSensorJob() {
        logger.info("Creating aggregateSensorJob...");
        return jobBuilderFactory.get("aggregateSensorJob")
                .start(aggregateSensorStep)
                .build();
    }
```
