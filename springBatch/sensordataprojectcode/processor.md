### **Explanation of `ItemProcessorForTextFile`**
This class is a **Spring Batch Item Processor** that processes sensor data for each day. It takes a `DailySensorData` object (raw sensor readings) and converts it into a `DailyAggregatedSensorData` object (with average, min, and max values).

---

### **Key Logic in `process()` Method**
1. **Extracts the date** from `DailySensorData`.
2. **Computes aggregate values** from sensor measurements:
   - **Average** (`avg`) → Sum of all values divided by count.
   - **Minimum** (`min`) → Smallest value.
   - **Maximum** (`max`) → Largest value.
3. **Creates and populates `DailyAggregatedSensorData`** with the computed values.
4. **Logs each step** for debugging.
5. **Returns the processed `DailyAggregatedSensorData` object**.

---

### **Full Code**
#### **1. `ItemProcessorForTextFile.java`**
```java
package com.practice.sensordata;

import java.util.List;
import java.util.OptionalDouble;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.springframework.batch.item.ItemProcessor;
import org.springframework.stereotype.Component;

@Component
public class ItemProcessorForTextFile implements ItemProcessor<DailySensorData, DailyAggregatedSensorData> {
    
    private final Logger logger = LogManager.getLogger(ItemProcessorForTextFile.class);

    @Override
    public DailyAggregatedSensorData process(DailySensorData item) throws Exception {
        
        logger.info("ItemProcessorForTextFile Started for Date: " + item.getDate());

        String date = item.getDate();
        List<Double> measurements = item.getMeasurements();

        if (measurements == null || measurements.isEmpty()) {
            logger.error("No measurements found for date: " + date);
            throw new IllegalArgumentException("Measurements list is empty for date: " + date);
        }

        double avg = measurements.stream().mapToDouble(Double::doubleValue).average().orElse(0.0);
        logger.info("Average calculated: " + avg);

        OptionalDouble min = measurements.stream().mapToDouble(Double::doubleValue).min();
        OptionalDouble max = measurements.stream().mapToDouble(Double::doubleValue).max();

        if (!min.isPresent() || !max.isPresent()) {
            logger.error("Failed to calculate min/max values for date: " + date);
            throw new IllegalStateException("Min/Max values could not be determined for date: " + date);
        }

        DailyAggregatedSensorData sensorData = new DailyAggregatedSensorData();
        sensorData.setDate(date);
        sensorData.setAvg(avg);
        sensorData.setMin(min.getAsDouble());
        sensorData.setMax(max.getAsDouble());

        logger.info("Processed Data: " + sensorData);

        return sensorData;
    }
}
```

---

#### **2. `DailyAggregatedSensorData.java` (Model Class)**
```java
package com.practice.sensordata;

import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
public class DailyAggregatedSensorData {
    private String date;
    private double avg;
    private double min;
    private double max;
}
```

---

### **Improvements & Features**
✔ **Handles empty measurement lists** with error checking.  
✔ Uses **OptionalDouble** to prevent errors if the list is empty.  
✔ **Logs each step** for debugging.  
✔ Uses **Lombok (`@Data`)** for clean and efficient code.  

This processor is useful in **Spring Batch workflows** for aggregating sensor data before further analysis or storage. 🚀
