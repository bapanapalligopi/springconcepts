```java
package com.practice.sensordata;

import java.util.HashMap;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.springframework.oxm.Marshaller;
import org.springframework.oxm.xstream.XStreamMarshaller;
import org.springframework.stereotype.Component;

import com.thoughtworks.xstream.security.ExplicitTypePermission;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

@Component
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class DailyAggregatedSensorData {
	private final static Logger logger = LogManager.getLogger(DailyAggregatedSensorData.class);
	public static final String ITEM_ROOT_ELEMENT_NAME = "daily-data";
	private String date;
	private Double min;
	private Double  max;
	private Double avg;
	public static Marshaller getMarshller() {
		// TODO Auto-generated method stub
		logger.info("DailyAggregatedSensorData started marsheler request.");
		XStreamMarshaller xStreamMarshaller = new XStreamMarshaller();
		logger.info("Marsheller object is created successfully.");
		HashMap<String, Class> alHashMap= new HashMap<>();
		alHashMap.put(ITEM_ROOT_ELEMENT_NAME, DailyAggregatedSensorData.class);
		alHashMap.put("min", Double.class);
		alHashMap.put("max", Double.class);
		alHashMap.put("avg", Double.class);
		alHashMap.put("date",String.class);
		logger.info("Aliasing Done Objects TO XML");
		
		ExplicitTypePermission typePermission = new ExplicitTypePermission(new Class[] { DailyAggregatedSensorData.class });
		
		logger.info("Successfully get the type persmissions");
		
		xStreamMarshaller.setAliases(alHashMap);
		xStreamMarshaller.getXStream().addPermission(typePermission);
		
		logger.info("xStreamMarshaller object have alias and type permissions");
		
		return xStreamMarshaller;
	}
}

```
