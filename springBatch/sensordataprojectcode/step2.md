```java
package com.practice.sensordata;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.springframework.batch.item.ItemReader;
import org.springframework.batch.item.xml.StaxEventItemReader;
import org.springframework.batch.item.xml.builder.StaxEventItemReaderBuilder;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.core.io.Resource;
import org.springframework.oxm.Unmarshaller;
import org.springframework.stereotype.Component;


@Component
public class XmlFileReaderForStep2 {
	private final static Logger logger= LogManager.getLogger(XmlFileReaderForStep2.class);
	
	@Value("classpath:HTE2NP.xml")
	private Resource xmlFilePath;
	@Autowired
	DailyAggregatedSensorData dailyAggregatedSensorData;
	
	@Bean
	public StaxEventItemReader<DailyAggregatedSensorData> getXmlReader(){
		
		logger.info("*************************xmlFileReaderForStep2 started*************************************");
		StaxEventItemReaderBuilder<DailyAggregatedSensorData> staxEventItemReaderBuilder= new StaxEventItemReaderBuilder<>();
		logger.info("xmlFileReaderForStep2 reader object is created");
		staxEventItemReaderBuilder.name("XmlFileReader for step2");
		logger.info("xmlFileReaderForStep2 reader name set");
		staxEventItemReaderBuilder.resource(xmlFilePath);
		logger.info("xmlFileReaderForStep2 reader resource set");
		staxEventItemReaderBuilder.unmarshaller((Unmarshaller) dailyAggregatedSensorData.getMarshller());
		logger.info("xmlFileReaderForStep2 reader unmarsheller  set");
		staxEventItemReaderBuilder.addFragmentRootElements(DailyAggregatedSensorData.ITEM_ROOT_ELEMENT_NAME);
		logger.info("xmlFileReaderForStep2 reader fragmentroot element set");
		logger.info("*************************xmlFileReaderForStep2 reader Completed*******************************");
		return staxEventItemReaderBuilder.build();
	}
}
package com.practice.sensordata;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.springframework.batch.item.ItemProcessor;
import org.springframework.stereotype.Component;

@Component
public class Step2ProcessorForAnamoly implements ItemProcessor<DailyAggregatedSensorData, DataAnomaly>{
	 // Threshold for low / high ratio to be considered anomaly vs normal
    private static final double THRESHOLD = 0.9;

	private final static Logger logger=LogManager.getLogger(Step2ProcessorForAnamoly.class);
	@Override
	public DataAnomaly process(DailyAggregatedSensorData item) throws Exception {
		// TODO Auto-generated method stub
		logger.info("***********************Step2ProcessorForAnamoly started***************************");
		DataAnomaly dataAnomaly = new DataAnomaly();
		logger.info("Data anomaly object is created");
		if ((item.getMin() / item.getAvg()) < THRESHOLD) {
			logger.info("WITH IN THRESH HOLD LIMIT ");
            return new DataAnomaly(item.getDate(), AnomalyType.MINIMUM, item.getMin());
        } else if ((item.getAvg() / item.getMax()) < THRESHOLD) {
        	logger.info("WITH IN THRESH HOLD LIMIT ");
            return new DataAnomaly(item.getDate(), AnomalyType.MAXIMUM, item.getMax());
        } else {
        	logger.info("OUTSIDE  THRESH HOLD LIMIT ");
            // Convention is to return null to filter item out and not pass it to the writer
            return null;
        }
		
	}

}
package com.practice.sensordata;


import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.springframework.batch.item.file.FlatFileItemReader;
import org.springframework.batch.item.file.FlatFileItemWriter;
import org.springframework.batch.item.file.builder.FlatFileItemReaderBuilder;
import org.springframework.batch.item.file.builder.FlatFileItemWriterBuilder;
import org.springframework.batch.item.file.transform.BeanWrapperFieldExtractor;
import org.springframework.batch.item.file.transform.DelimitedLineAggregator;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.core.io.FileSystemResource;
import org.springframework.core.io.Resource;
import org.springframework.core.io.ResourceLoader;
import org.springframework.stereotype.Component;

@Component
public class TextFileItemWriter {
	
	private static final Logger LOGGER= LogManager.getLogger(TextFileItemWriter.class);
	@Autowired
	ResourceLoader resourceLoader;
	
	@Bean
	public FlatFileItemWriter<DataAnomaly> flatFileItemReader(){
		
		FileSystemResource resource = new FileSystemResource("HTE2NP-anomalies.csv");
		FlatFileItemWriterBuilder<DataAnomaly> flatFileItemWriterBuilder= new FlatFileItemWriterBuilder<>();
		// Configure line aggregator
		DelimitedLineAggregator<DataAnomaly> lineAggregator = new DelimitedLineAggregator<>();
		lineAggregator.setDelimiter(",");
		BeanWrapperFieldExtractor<DataAnomaly> beanWrapperFieldExtractor = new BeanWrapperFieldExtractor<>();
		beanWrapperFieldExtractor.setNames(new String[] {"date","type","value"});
		lineAggregator.setFieldExtractor(beanWrapperFieldExtractor);
		
		LOGGER.info("**********TextFileItemWriter started for step2*******************");
		LOGGER.info("TextFileItemWriter Object created succesfully");
		flatFileItemWriterBuilder.name("Writer for step2");

		LOGGER.info("TextFileItemWriter Object writer name set");
		flatFileItemWriterBuilder.resource(resource);
		LOGGER.info("TextFileItemWriter Object resource set");
		flatFileItemWriterBuilder.lineAggregator(lineAggregator);
		LOGGER.info("TextFileItemWriter Object line aggregator set");
		LOGGER.info("***************Writer for Step2 Completed**************************");
		return flatFileItemWriterBuilder.build();
	}
}
 @Bean
    @Qualifier("anamolyStep")
    public Step anamaolyStep() {
    	
    	logger.info("Defining anamolyStep...");
    	return new StepBuilderFactory(getJobRepository(), getTransactionManager())
    			.get("anamoly step")
    			.<DailyAggregatedSensorData,DataAnomaly>chunk(1)
    			.reader(xmlFileReaderForStep2.getXmlReader())
    			.processor(step2ProcessorForAnamoly)
    			.writer(textFileItemWriter.flatFileItemReader())
    			.build();
@Bean
    public Job aggregateSensorJob() {
        logger.info("Creating aggregateSensorJob...");
        return jobBuilderFactory.get("hai")
        		.incrementer(new RunIdIncrementer())// Allows re-running the job
                .start(aggregateSensorStep)
                .next(anamaolyStep())
                .build();
@Override
	public void run(String... args) throws Exception {
		// TODO Auto-generated method stub
		logger.info("Starting Batch Job...");
		JobParameters jobParameters = new JobParametersBuilder().addDate("date",new Date()).toJobParameters();
		org.springframework.batch.core.JobExecution  jobExecution= jobLauncher.run(temperatureSensorRootConfiguration.aggregateSensorJob(), jobParameters);
		
		logger.info("Batch Job Status: " + jobExecution.getExitStatus());

	}
```
