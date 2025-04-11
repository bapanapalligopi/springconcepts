# Creating XLSX Files in Java Using Apache POI

Apache POI is a popular Java library for working with Microsoft Office documents, including Excel files (XLSX format). Here's how to create an XLSX file:

## Prerequisites

First, add the Apache POI dependencies to your project. For Maven, add these to your pom.xml:

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>5.2.5</version> <!-- or latest version -->
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.2.5</version> <!-- or latest version -->
</dependency>
```

## Basic Example

Here's a simple example to create an XLSX file:

```java
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.FileOutputStream;
import java.io.IOException;

public class ExcelCreator {
    public static void main(String[] args) {
        // Create a new workbook
        Workbook workbook = new XSSFWorkbook();
        
        // Create a sheet
        Sheet sheet = workbook.createSheet("Employee Data");
        
        // Create header row
        Row headerRow = sheet.createRow(0);
        String[] headers = {"ID", "Name", "Department", "Salary"};
        
        for (int i = 0; i < headers.length; i++) {
            Cell cell = headerRow.createCell(i);
            cell.setCellValue(headers[i]);
        }
        
        // Add some data
        Object[][] data = {
            {1, "John Doe", "IT", 75000},
            {2, "Jane Smith", "HR", 65000},
            {3, "Bob Johnson", "Finance", 80000}
        };
        
        for (int i = 0; i < data.length; i++) {
            Row row = sheet.createRow(i + 1);
            for (int j = 0; j < data[i].length; j++) {
                Cell cell = row.createCell(j);
                if (data[i][j] instanceof String) {
                    cell.setCellValue((String) data[i][j]);
                } else if (data[i][j] instanceof Integer) {
                    cell.setCellValue((Integer) data[i][j]);
                } else if (data[i][j] instanceof Double) {
                    cell.setCellValue((Double) data[i][j]);
                }
            }
        }
        
        // Auto-size columns
        for (int i = 0; i < headers.length; i++) {
            sheet.autoSizeColumn(i);
        }
        
        // Write the output to a file
        try (FileOutputStream outputStream = new FileOutputStream("employees.xlsx")) {
            workbook.write(outputStream);
            System.out.println("Excel file created successfully!");
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                workbook.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## Advanced Features

### Cell Styling

```java
// Create a CellStyle with a font
CellStyle headerStyle = workbook.createCellStyle();
Font headerFont = workbook.createFont();
headerFont.setBold(true);
headerFont.setFontHeightInPoints((short) 12);
headerFont.setColor(IndexedColors.BLUE.getIndex());
headerStyle.setFont(headerFont);

// Apply the style to header cells
for (int i = 0; i < headers.length; i++) {
    Cell cell = headerRow.createCell(i);
    cell.setCellValue(headers[i]);
    cell.setCellStyle(headerStyle);
}
```

### Date Handling

```java
// Create a date cell style
CellStyle dateCellStyle = workbook.createCellStyle();
CreationHelper createHelper = workbook.getCreationHelper();
dateCellStyle.setDataFormat(
    createHelper.createDataFormat().getFormat("mm/dd/yyyy"));

// Create a cell with a date
Row dateRow = sheet.createRow(sheet.getLastRowNum() + 1);
Cell dateCell = dateRow.createCell(0);
dateCell.setCellValue(new Date());
dateCell.setCellStyle(dateCellStyle);
```

### Formulas

```java
// Add a formula to calculate sum of salaries
Row formulaRow = sheet.createRow(sheet.getLastRowNum() + 1);
Cell formulaCell = formulaRow.createCell(3); // Salary column
formulaCell.setCellFormula("SUM(D2:D4)"); // Sum of D2 to D4
```

## Best Practices

1. Always close resources (Workbook, OutputStream) in finally blocks or use try-with-resources
2. For large files, use `SXSSFWorkbook` instead of `XSSFWorkbook` for better memory management
3. Auto-size columns after adding all data
4. Consider using helper methods for repetitive tasks like cell creation

This should give you a good starting point for creating XLSX files with Apache POI in Java!
