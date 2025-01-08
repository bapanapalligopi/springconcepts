Here's the code you requested, with the same structure and functionality:

```java
package com.example.demo.config;

import org.springframework.batch.core.Job;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.configuration.annotation.EnableBatchProcessing;
import org.springframework.batch.core.configuration.annotation.JobBuilderFactory;
import org.springframework.batch.core.job.builder.JobBuilder;
import org.springframework.batch.core.repository.JobRepository;
import org.springframework.batch.core.step.builder.StepBuilder;
import org.springframework.batch.item.data.RepositoryItemWriter;
import org.springframework.batch.item.file.FlatFileItemReader;
import org.springframework.batch.item.file.LineMapper;
import org.springframework.batch.item.file.mapping.BeanWrapperFieldSetMapper;
import org.springframework.batch.item.file.mapping.DefaultLineMapper;
import org.springframework.batch.item.file.transform.DelimitedLineTokenizer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.FileSystemResource;
import org.springframework.transaction.PlatformTransactionManager;

import com.example.demo.Repo.CustomerRepo;
import com.example.demo.modesl.Customer;
import com.example.demo.modesl.CustomerProcessor;

@Configuration
//@EnableBatchProcessing
public class BatchConfig {

	@Autowired
	JobRepository jobRepository;
	
	@Autowired
	CustomerRepo customerRepo;

	@Autowired
    private PlatformTransactionManager transactionManager;

	// Create item reader to read data from the source (CSV file)
	@Bean
	public FlatFileItemReader<Customer> csvFilereader() {
		// Create a reader object
		FlatFileItemReader<Customer> itemReader = new FlatFileItemReader<>();
		
		// Take the data from the given path
		itemReader.setResource(new FileSystemResource("src/main/resources/customers_data.csv"));
		// Give the name for the reader
		itemReader.setName("csvFileReader");
		// Skip the first line from CSV file like header
		itemReader.setLinesToSkip(1);
		// Define line mapper, read one line from CSV and represent that line as a Customer object
		itemReader.setLineMapper(lineMapper());
		
		// Return the item reader to the job
		return itemReader;
	}

	// Line mapper for item reader csvFilereader
	private LineMapper<Customer> lineMapper() {
		// Create a line mapper by default
		DefaultLineMapper<Customer> lineMapper = new DefaultLineMapper<>();
		
		// Data is separated by comma (CSV format)
		DelimitedLineTokenizer lineTokenizer = new DelimitedLineTokenizer();
		lineTokenizer.setDelimiter(",");
		// Set strict to false if columns are missing; missing values will be set to null
		lineTokenizer.setStrict(false);
		// Set the file attributes in CSV
		lineTokenizer.setNames("id", "firstname", "lastname", "gender", "contactNo", "country", "dob");
		
		// Convert to bean object
		BeanWrapperFieldSetMapper<Customer> fieldSetMapper = new BeanWrapperFieldSetMapper<>();
		fieldSetMapper.setTargetType(Customer.class);
		
		// Set the line mapper properties
		lineMapper.setFieldSetMapper(fieldSetMapper);
		lineMapper.setLineTokenizer(lineTokenizer);
		
		// Return the line mapper
		return lineMapper;
	}

	// Create an item processor
	@Bean
	public CustomerProcessor customerProcessor() {
		return new CustomerProcessor();
	}

	// Create an item writer
	@Bean
	public RepositoryItemWriter<Customer> customerItemWriter() {
		// Create a repositoryItemWriter object
		RepositoryItemWriter<Customer> repositoryItemWriter = new RepositoryItemWriter<>();
		// Set the repository
		repositoryItemWriter.setRepository(customerRepo);
		// Insert the data using save method
		repositoryItemWriter.setMethodName("save");
		
		// Return the writer
		return repositoryItemWriter;
	}

	// Create a Step
	@Bean
	public Step step() {
		// Directly create a Step using the StepBuilder
		return new StepBuilder("step", jobRepository)
                .<Customer, Customer>chunk(10)  // Define chunk size
                .reader(csvFilereader())       // Define your reader
                .processor(customerProcessor()) // Define your processor
                .writer(customerItemWriter())  // Define your writer
                .transactionManager(transactionManager) // Use the transaction manager
                .build();
	}

	// Create a Job object
	@Bean
	public Job job() {
		// Create a job with the steps
		return new JobBuilderFactory(jobRepository)
				.get("cdv") // Job name
				.start(step()) // Starting step
				.build(); // Finalize and return the job
	}
}
```

### Explanation:
This code configures a Spring Batch job that reads data from a CSV file, processes it, and writes the result to a database using Spring's repository mechanism. Here's a breakdown of the process:

1. **Item Reader (`csvFilereader`)**: 
   - The `FlatFileItemReader<Customer>` reads the customer data from a CSV file located at `"src/main/resources/customers_data.csv"`.
   - It uses a `LineMapper` to map each line from the CSV to a `Customer` object.

2. **Line Mapper**: 
   - A `DelimitedLineTokenizer` is used to tokenize the CSV file, where each line is separated by commas.
   - A `BeanWrapperFieldSetMapper` is used to map each field in the CSV line to the corresponding fields in the `Customer` class.

3. **Item Processor (`CustomerProcessor`)**:
   - The `CustomerProcessor` is responsible for processing each `Customer` object before writing it to the database. In this case, it just returns the item as is, but you can add custom processing logic here if needed.

4. **Item Writer (`customerItemWriter`)**:
   - A `RepositoryItemWriter<Customer>` is used to write the processed `Customer` objects to the database.
   - It calls the `save` method of the `CustomerRepo` repository to persist each customer.

5. **Step**: 
   - A `Step` is created using a `StepBuilder`. It specifies the chunk size of 10, meaning 10 records are read, processed, and written at a time.
   - The reader, processor, and writer are wired into the step, and the `transactionManager` is used to manage transactions for the step.

6. **Job**: 
   - A `Job` is created with a single step (`step()`), and it is named `"cdv"`.
   - The job is built using the `JobBuilderFactory` and returned as a bean for execution.

### Purpose:
This configuration sets up a Spring Batch job that can be run to process data from a CSV file. It reads data in chunks, processes it, and writes it to a database in a transactional manner.

Let me know if you need any further clarifications or adjustments!
