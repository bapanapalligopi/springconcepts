Here is an example implementation of a `Tasklet` in Spring Batch. The tasklet will simulate processing records (for example, reading and processing data from a file or database). In this example, we'll create a tasklet that processes a simple list of numbers and stops after processing the entire list.

### Example: Tasklet to Process Numbers

```java
import org.springframework.batch.core.StepContribution;
import org.springframework.batch.core.scope.context.ChunkContext;
import org.springframework.batch.core.tasklet.Tasklet;
import org.springframework.batch.repeat.RepeatStatus;

public class NumberProcessingTasklet implements Tasklet {

    private int currentIndex = 0;
    private final int[] numbers = {1, 2, 3, 4, 5};  // Sample data to be processed

    @Override
    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
        // Check if all numbers are processed
        if (currentIndex >= numbers.length) {
            return RepeatStatus.FINISHED;  // Task is finished, no more work to do
        }

        // Process the current number
        int number = numbers[currentIndex];
        System.out.println("Processing number: " + number);

        // Increment the index for the next execution
        currentIndex++;

        // Update the step contribution (optional)
        contribution.incrementReadCount(1);  // Track how many items have been processed

        return RepeatStatus.CONTINUABLE;  // Indicate that more work needs to be done
    }
}
```

### Explanation:
1. **State Variables**:
   - `currentIndex`: Keeps track of the current position in the `numbers` array.
   - `numbers`: A simple array of integers that we are processing in our tasklet. In a real-world scenario, this could be reading data from a file, database, etc.

2. **execute Method**:
   - The method processes each number in the `numbers` array sequentially.
   - It checks whether the tasklet has finished processing all numbers. If so, it returns `RepeatStatus.FINISHED` to indicate the task is complete.
   - If there are more numbers to process, it processes the current number, increments the `currentIndex`, and returns `RepeatStatus.CONTINUABLE`, indicating that the tasklet should continue running in the next iteration.

3. **StepContribution**:
   - In this example, we use the `contribution.incrementReadCount(1)` method to increment the "read count", which can be helpful in tracking progress, especially for logging or debugging purposes.

### Step Configuration in Spring Batch:

In a Spring Batch job configuration, this tasklet can be used in a step like this:

```java
import org.springframework.batch.core.Step;
import org.springframework.batch.core.configuration.annotation.JobBuilderFactory;
import org.springframework.batch.core.configuration.annotation.StepBuilderFactory;
import org.springframework.batch.core.launch.JobLauncher;
import org.springframework.batch.core.launch.support.RunIdIncrementer;
import org.springframework.batch.core.Job;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class BatchConfig {

    @Bean
    public Job processNumbersJob(JobBuilderFactory jobBuilderFactory,
                                 StepBuilderFactory stepBuilderFactory) {
        return jobBuilderFactory.get("processNumbersJob")
                .incrementer(new RunIdIncrementer())
                .start(processNumbersStep(stepBuilderFactory))
                .build();
    }

    @Bean
    public Step processNumbersStep(StepBuilderFactory stepBuilderFactory) {
        return stepBuilderFactory.get("processNumbersStep")
                .tasklet(new NumberProcessingTasklet())
                .build();
    }
}
```

### Explanation of the Batch Configuration:
1. **Job**:
   - The `processNumbersJob` defines the batch job and uses a simple incrementer to create a new job instance on each execution.
   - It starts with the `processNumbersStep` which uses the `NumberProcessingTasklet`.

2. **Step**:
   - The `processNumbersStep` is defined using `tasklet(new NumberProcessingTasklet())`, which tells Spring Batch to execute the `NumberProcessingTasklet` as part of this step.

### How It Works:
- When the job is executed, the `NumberProcessingTasklet` will run through the numbers array.
- It will print the numbers one by one, and after processing all numbers, it will return `RepeatStatus.FINISHED`, indicating the task is complete.

### Output:
```plaintext
Processing number: 1
Processing number: 2
Processing number: 3
Processing number: 4
Processing number: 5
```

The job completes when the tasklet processes all the numbers, and `RepeatStatus.FINISHED` is returned.

### Notes:
- This is a very simple example. In real applications, tasklets might process more complex data sources (e.g., files, databases) or perform various types of transformations.
- You can also implement additional logic for error handling, retries, and logging within the tasklet.
