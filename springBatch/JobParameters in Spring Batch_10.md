### JobParameters in Spring Batch

In Spring Batch, **JobParameters** are used to distinguish between different executions of the same batch job. They provide a way to uniquely identify each **JobInstance**, even though the underlying job may remain the same. A **JobInstance** is associated with a set of **JobParameters**, and these parameters can be used for identification, reference data, or execution control.

### Key Concepts of **JobParameters**:

1. **Identification of JobInstance**:
   - **JobParameters** are used to uniquely identify a **JobInstance**. For instance, if the same job runs on different dates (e.g., `EndOfDayJob` on January 1st and January 2nd), the **JobParameters** (e.g., `date=2025-01-01` and `date=2025-01-02`) help create distinct **JobInstances**.
   - In this scenario, even though the job (`EndOfDayJob`) is the same, each execution on a different date will be tracked as a separate **JobInstance** because of the different **JobParameters**.

2. **Distinguishing Different Executions**:
   - Each time a job is launched, **JobParameters** are passed to the job, and they will be used to check if an existing **JobInstance** already exists with the same parameters. If the parameters are the same, the system will identify it as the same **JobInstance**, and it can either resume from where it left off (if restartable) or be re-executed.
   - For example, if the job parameters are set as `date=2025-01-01`, the batch system will create a **JobInstance** for January 1st. If the job is re-executed on January 2nd with the same parameter (`date=2025-01-01`), it will be considered the same **JobInstance** (even though it's a new execution attempt).

3. **JobParameters as Reference Data**:
   - **JobParameters** are not just for identification; they can also serve as reference data during the job execution. For example, a **JobParameter** might indicate a specific file to process, a batch size, or other business-specific configurations that guide the processing logic during execution.

4. **Optional Contribution to JobInstance Identity**:
   - By default, **JobParameters** are used to identify a **JobInstance**. However, Spring Batch provides flexibility to allow certain parameters to be excluded from contributing to the **JobInstance**'s identity.
   - For example, some parameters might only be needed for processing within the job (like configuration parameters) but should not impact the identity of the **JobInstance**. These parameters can be marked as non-identifying.

### How **JobParameters** Work:

- **Default Behavior**: By default, **JobParameters** are used to define the identity of the **JobInstance**. Each **JobInstance** has a unique set of parameters that distinguish it from others, even if the same job is executed multiple times.

- **Customizing JobParameter Contribution**: You can customize which parameters contribute to the identity of the **JobInstance**. For example, parameters that are purely for internal use (such as the name of a file to be processed) may be excluded from the identity calculation.

#### Example: Creating JobParameters

```java
import org.springframework.batch.core.JobParameters;
import org.springframework.batch.core.JobParametersBuilder;

public class JobParameterExample {

    public JobParameters createJobParameters(String date) {
        return new JobParametersBuilder()
                .addString("date", date)  // Identifier
                .addString("fileName", "transactions.csv")  // Reference data (but not part of identity)
                .toJobParameters();
    }
}
```

In this example:
- **`date`** is a **JobParameter** that would contribute to the **JobInstance** identity, as the job is expected to run daily.
- **`fileName`** is a parameter used as reference data for the job's processing logic (e.g., identifying which file to process), but it doesn't contribute to the uniqueness of the **JobInstance**.

#### Example: JobLauncher with JobParameters

```java
import org.springframework.batch.core.Job;
import org.springframework.batch.core.launch.JobLauncher;
import org.springframework.batch.core.JobParameters;
import org.springframework.batch.core.JobParametersBuilder;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class BatchJobRunner {

    @Autowired
    private JobLauncher jobLauncher;

    @Autowired
    private Job exampleJob; // Example job bean

    public void runJob(String date) throws Exception {
        JobParameters jobParameters = new JobParametersBuilder()
            .addString("date", date) // Set the date parameter for uniqueness
            .toJobParameters();

        jobLauncher.run(exampleJob, jobParameters); // Launch the job with the parameters
    }
}
```

In this example, the job is launched with **JobParameters**, which includes the `date` parameter. This `date` parameter will define the identity of the **JobInstance**, allowing the job to track and manage the execution uniquely for that day.

---

### JobParameter Types

**JobParameters** support different types of data, and each type is stored in a corresponding **JobParameter**. The following types are supported:

1. **String**:
   - Can store any string value. Useful for textual data like file names, job identifiers, etc.
   
   Example: `addString("fileName", "data.csv")`

2. **Long**:
   - Stores a long integer value, which can be useful for IDs, timestamps, or other large numeric values.
   
   Example: `addLong("jobId", 123456L)`

3. **Double**:
   - Stores a double-precision floating point number. It is useful for values like percentages or financial amounts.
   
   Example: `addDouble("discountRate", 5.75)`

4. **Date**:
   - Stores a date value. This is commonly used for date-related parameters like execution time, batch date, etc.
   
   Example: `addDate("startDate", new Date())`

5. **Map**:
   - You can also store a **Map** of parameters. This is especially useful if you have a large number of parameters that should be grouped together.

---

### Example Use Case: Batch Job with Date Parameter

Imagine a batch job that processes customer orders for a specific date. Each time the job runs, it will process a unique set of orders for that day, and the date will be passed as a **JobParameter** to identify the specific **JobInstance**.

#### Job Definition:

```java
@Bean
public Job processOrdersJob(JobBuilderFactory jobBuilderFactory, Step step1) {
    return jobBuilderFactory.get("processOrdersJob")
                            .start(step1)
                            .build();
}
```

#### JobLauncher with Parameters:

```java
public void runJobForSpecificDate(String date) throws Exception {
    JobParameters jobParameters = new JobParametersBuilder()
                                    .addString("date", date)
                                    .toJobParameters();

    jobLauncher.run(processOrdersJob, jobParameters);
}
```

Here, each time the job is run with a different **date** parameter (e.g., `date=2025-01-01`, `date=2025-01-02`), a new **JobInstance** will be created, and the system will track the job's execution separately for each date. 

---

### Summary of **JobParameters**:

- **JobParameters** are used to uniquely identify a **JobInstance**. The parameters passed when launching a job determine whether the job should be treated as a new execution or a continuation of an existing execution.
- **JobParameters** allow you to include both identifying data (e.g., `date`) and reference data (e.g., `fileName`) for the execution of the job.
- The **JobParameters** object can store multiple types of data, including `String`, `Long`, `Date`, `Double`, etc.
- You can control which parameters contribute to the identity of the **JobInstance** by making them part of the identity or excluding them as needed.
