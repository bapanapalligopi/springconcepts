### Explanation of `SensorDataTextLineMapper.java`

This Java class is a **Spring Batch** component that implements the `LineMapper<DailySensorData>` interface. Its purpose is to **map a line of text data** from an input file to a `DailySensorData` object.

---

### **Class Breakdown:**
1. **Imports & Annotations**
   - The class imports necessary Java utilities (`ArrayList`, `Iterator`), Spring validation, logging (`LogManager`, `Logger`), and Spring Batch (`LineMapper`).
   - It is marked as a **Spring component** using `@Component`, making it a managed Spring bean.
   - The `@NoArgsConstructor` annotation (from Lombok) automatically generates a no-argument constructor.

2. **Logger Initialization**
   ```java
   private final Logger logger= LogManager.getLogger(SensorDataTextLineMapper.class);
   ```
   - A logger from `Log4j` is initialized for logging debug information.

3. **Method: `mapLine(String line, int lineNumber)`**
   - This method is responsible for **parsing** a line from a text file and mapping it into a `DailySensorData` object.

---

### **Logic Flow of `mapLine` Method**
#### **1. Input Format (Assumption)**
Each line in the file follows this format:
   ```
   01-06-2015:76.53,76.81,76.00,75.30,74.91,74.05,73.93,74.02,74.87
   ```
   - The first part (`01-06-2015`) represents the **date**.
   - The second part (`76.53,76.81,76.00,...`) represents **sensor measurements** (comma-separated).

---

#### **2. Step-by-Step Execution**

**Step 1: Split Line by `:`**
```java
String[] splittedLineData = line.split(":");
```
- Splits the input string into two parts:
  - `splittedLineData[0]`: Date → `"01-06-2015"`
  - `splittedLineData[1]`: Sensor readings → `"76.53,76.81,76.00,75.30,74.91,74.05,73.93,74.02,74.87"`

---

**Step 2: Assign Date to `DailySensorData`**
```java
DailySensorData dailySensorData = new DailySensorData();
dailySensorData.setDate(splittedLineData[0]);
```
- Creates an instance of `DailySensorData`.
- Assigns the extracted date (`splittedLineData[0]`).

---

**Step 3: Split Sensor Values by `,`**
```java
String[] splittedWithComma = splittedLineData[1].split(",");
```
- This splits the second part of the line (sensor readings) into an array of strings.

Example:
```java
["76.53", "76.81", "76.00", "75.30", "74.91", "74.05", "73.93", "74.02", "74.87"]
```

---

**Step 4: Convert String Array to `ArrayList<Double>`**
```java
ArrayList<Double> commaSeparatedValue = new ArrayList<Double>();
for(String commaValueString : splittedWithComma) {
    commaSeparatedValue.add(Double.parseDouble(commaValueString));
}
```
- Initializes an empty `ArrayList<Double>`.
- Iterates through the `splittedWithComma` array.
- Converts each **string value** into a **double** and adds it to the list.

---

**Step 5: Assign Sensor Readings to `DailySensorData`**
```java
dailySensorData.setMeasurments(commaSeparatedValue);
```
- Assigns the list of sensor values to the `DailySensorData` object.

---

**Step 6: Return the Mapped Object**
```java
return dailySensorData;
```
- Returns the fully populated `DailySensorData` object.

---

### **Final Output (`DailySensorData` Object)**
For an input line:
```
01-06-2015:76.53,76.81,76.00,75.30,74.91,74.05,73.93,74.02,74.87
```
The method will create and return:
```java
DailySensorData {
    date = "01-06-2015",
    measurments = [76.53, 76.81, 76.00, 75.30, 74.91, 74.05, 73.93, 74.02, 74.87]
}
```

---

### **Logging (`logger.info`)**
Throughout the process, **log messages** help track the execution:
- `"LINE MAPPER STARTED FOR MAP THE LINE"`
- `"SPLITTED WITH : DONE"`
- `"DATE ASSIGNED"`
- `"SPLITTED WITH , DONE"`
- `"LIST OF VALUES ARE ASSIGNED"`

Here’s the full implementation of your **SensorDataTextLineMapper** along with the **DailySensorData** class:  

---

### **1. `SensorDataTextLineMapper.java`**
```java
package com.practice.sensordata;

import java.util.ArrayList;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.springframework.batch.item.file.LineMapper;
import org.springframework.stereotype.Component;

import lombok.NoArgsConstructor;

@Component
@NoArgsConstructor
public class SensorDataTextLineMapper implements LineMapper<DailySensorData> {
    
    private final Logger logger = LogManager.getLogger(SensorDataTextLineMapper.class);

    @Override
    public DailySensorData mapLine(String line, int lineNumber) throws Exception {
        logger.info("LINE MAPPER STARTED FOR MAPPING THE LINE");

        String[] splittedLineData = line.split(":");
        if (splittedLineData.length < 2) {
            logger.error("Invalid line format at line number: " + lineNumber);
            throw new IllegalArgumentException("Invalid line format: " + line);
        }

        logger.info("SPLITTED WITH : DONE");

        DailySensorData dailySensorData = new DailySensorData();
        dailySensorData.setDate(splittedLineData[0].trim());
        logger.info("DATE ASSIGNED: " + dailySensorData.getDate());

        String[] splittedWithComma = splittedLineData[1].split(",");
        logger.info("SPLITTED WITH , DONE");

        ArrayList<Double> measurements = new ArrayList<>();
        try {
            for (String value : splittedWithComma) {
                measurements.add(Double.parseDouble(value.trim()));
            }
        } catch (NumberFormatException e) {
            logger.error("Error parsing measurement values at line number: " + lineNumber, e);
            throw e;
        }

        dailySensorData.setMeasurements(measurements);
        logger.info("LIST OF VALUES ASSIGNED: " + measurements);
        return dailySensorData;
    }
}
```

---

### **2. `DailySensorData.java`**
This is the **model class** representing the data structure for daily sensor readings.

```java
package com.practice.sensordata;

import java.util.List;

import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
public class DailySensorData {
    private String date;
    private List<Double> measurements;
}
```

---

### **Explanation**
1. **`SensorDataTextLineMapper` (Mapper Class)**
   - Reads a **single line** from a text file.
   - Splits the data by `:` to separate the date and sensor values.
   - Splits the second part by `,` to extract sensor readings.
   - Converts sensor readings from `String` to `Double`.
   - Stores the data in a `DailySensorData` object.
   - Uses **Log4j logging** to track progress.
   - Includes error handling for **invalid formats**.

2. **`DailySensorData` (Data Model)**
   - Stores:
     - **Date** (`String`).
     - **Sensor Measurements** (`List<Double>`).
   - Uses **Lombok (`@Data`, `@NoArgsConstructor`)** for automatic getter/setter generation.

---

### **Example Input & Output**
#### **Input Line:**
```
01-06-2015:76.53,76.81,76.00,75.30,74.91,74.05,73.93,74.02,74.87
```

#### **Mapped Output (`DailySensorData` object)**
```java
DailySensorData {
    date = "01-06-2015",
    measurements = [76.53, 76.81, 76.00, 75.30, 74.91, 74.05, 73.93, 74.02, 74.87]
}
```

---
