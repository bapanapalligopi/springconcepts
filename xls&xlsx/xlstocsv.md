# **Complete Spring Batch Tasklet with Enhanced Validation and Logging**

Here's the full implementation with all requested validations, detailed error reporting, and comprehensive logging:

```java
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.ss.usermodel.*;
import org.springframework.batch.core.StepContribution;
import org.springframework.batch.core.scope.context.ChunkContext;
import org.springframework.batch.core.step.tasklet.Tasklet;
import org.springframework.batch.repeat.RepeatStatus;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.*;
import java.text.SimpleDateFormat;
import java.util.*;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

@Component
public class XlsToCsvConversionTasklet implements Tasklet {

    private static final Logger logger = LoggerFactory.getLogger(XlsToCsvConversionTasklet.class);
    
    @Value("${input.file.path}")
    private String inputFilePath;

    @Value("${output.file.path}")
    private String outputFilePath;

    @Value("${error.file.path}")
    private String errorFilePath;

    @Value("${report.file.path}")
    private String reportFilePath;

    @Value("${expected.column.count}")
    private int expectedColumnCount;

    @Value("${expected.column.headers}")
    private String[] expectedHeaders;

    @Value("${expected.column.formats}")
    private String[] expectedFormats;

    @Override
    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
        ConversionResult result = new ConversionResult();
        
        try {
            logger.info("Starting XLS to CSV conversion for file: {}", inputFilePath);
            
            // 1. File existence check
            File inputFile = new File(inputFilePath);
            if (!inputFile.exists()) {
                String error = "Input file not found: " + inputFilePath;
                logger.error(error);
                throw new ValidationException(error);
            }

            // 2. Empty file check
            if (inputFile.length() == 0) {
                String error = "Input file is empty";
                logger.error(error);
                throw new ValidationException(error);
            }

            // 3. Perform conversion with validations
            result = convertXlsToCsvWithValidation();

            // 4. Check if only headers exist
            if (result.totalRows == 1) {
                String error = "File contains only headers, no data rows";
                logger.error(error);
                throw new ValidationException(error);
            }

            // 5. Final validation - Check if any data was converted
            if (result.convertedRows == 0) {
                String error = "No valid data found to convert after validation";
                logger.error(error);
                throw new ValidationException(error);
            }

            generateReport(result);
            logger.info("Successfully converted {} rows", result.convertedRows);

            contribution.incrementWriteCount(result.convertedRows);
            chunkContext.getStepContext().getStepExecution().getJobExecution()
                .getExecutionContext().put("conversionReport", result);

        } catch (ValidationException e) {
            logger.error("Validation failed: {}", e.getMessage());
            result.errorMessages.add(e.getMessage());
            generateReport(result);
            throw e;
        } catch (Exception e) {
            logger.error("Conversion failed: {}", e.getMessage(), e);
            result.errorMessages.add("Fatal error: " + e.getMessage());
            generateReport(result);
            throw e;
        }

        return RepeatStatus.FINISHED;
    }

    public class ConversionResult {
        public int totalRows;
        public int convertedRows;
        public List<Integer> errorRows = new ArrayList<>();
        public List<String> errorMessages = new ArrayList<>();
        public Map<String, Integer> errorTypes = new HashMap<>();
        public List<String> missingColumns = new ArrayList<>();
        public List<String> invalidFormatColumns = new ArrayList<>();
        public boolean validationPassed;
    }

    public class ValidationException extends RuntimeException {
        public ValidationException(String message) {
            super(message);
        }
    }

    private ConversionResult convertXlsToCsvWithValidation() throws IOException, ValidationException {
        ConversionResult result = new ConversionResult();
        
        try (InputStream inp = new FileInputStream(inputFilePath);
             HSSFWorkbook workbook = new HSSFWorkbook(inp);
             PrintWriter writer = new PrintWriter(new FileWriter(outputFilePath));
             PrintWriter errorWriter = new PrintWriter(new FileWriter(errorFilePath))) {

            Sheet sheet = workbook.getSheetAt(0);
            result.totalRows = sheet.getLastRowNum() + 1;
            logger.debug("Found {} rows in input file", result.totalRows);

            // 1. Column Count Validation
            Row headerRow = sheet.getRow(0);
            if (headerRow == null) {
                throw new ValidationException("No header row found in the input file");
            }

            int actualColumnCount = headerRow.getLastCellNum();
            if (actualColumnCount != expectedColumnCount) {
                result.missingColumns = findMissingColumns(headerRow);
                String error = String.format("Column count mismatch. Expected: %d, Actual: %d. Missing columns: %s",
                    expectedColumnCount, 
                    actualColumnCount,
                    result.missingColumns);
                logger.error(error);
                throw new ValidationException(error);
            }

            // 2. Header Name Validation
            List<String> headerValidationErrors = validateHeaders(headerRow);
            if (!headerValidationErrors.isEmpty()) {
                String error = "Header validation failed: " + String.join(", ", headerValidationErrors);
                logger.error(error);
                throw new ValidationException(error);
            }

            // 3. Column Format Validation
            result.invalidFormatColumns = validateColumnFormats(sheet);
            if (!result.invalidFormatColumns.isEmpty()) {
                String error = "Format validation failed for columns: " + String.join(", ", result.invalidFormatColumns);
                logger.error(error);
                throw new ValidationException(error);
            }

            // Write CSV headers
            List<String> headers = new ArrayList<>();
            for (int i = 0; i < expectedColumnCount; i++) {
                headers.add(escapeForCsv(headerRow.getCell(i).getStringCellValue()));
            }
            writer.println(String.join(",", headers));
            errorWriter.println(String.join(",", headers) + ",Error_Type,Error_Details");

            // Process data rows
            for (int i = 1; i <= sheet.getLastRowNum(); i++) {
                Row row = sheet.getRow(i);
                if (row == null) {
                    handleError(result, i, "EMPTY_ROW", "Complete row is empty");
                    errorWriter.println(buildErrorLine(headers.size(), "EMPTY_ROW", "Row " + (i+1) + " is completely empty"));
                    continue;
                }

                StringBuilder csvLine = new StringBuilder();
                boolean isRowEmpty = true;
                boolean hasNullValues = false;
                boolean hasFormatErrors = false;
                List<String> errorDetails = new ArrayList<>();

                for (int j = 0; j < headers.size(); j++) {
                    Cell cell = row.getCell(j, Row.MissingCellPolicy.CREATE_NULL_AS_BLANK);
                    String cellValue;

                    // Format validation
                    if (!validateCellFormat(cell, expectedFormats[j])) {
                        hasFormatErrors = true;
                        errorDetails.add(String.format("%s: Expected %s got %s", 
                            headers.get(j), 
                            expectedFormats[j], 
                            getCellTypeName(cell.getCellType())));
                    }

                    cellValue = getCellValueAsString(cell);

                    if (cellValue == null || cellValue.trim().isEmpty()) {
                        cellValue = "NULL";
                        hasNullValues = true;
                    } else {
                        isRowEmpty = false;
                    }

                    csvLine.append(escapeForCsv(cellValue));
                    if (j < headers.size() - 1) csvLine.append(",");
                }

                if (hasFormatErrors) {
                    handleError(result, i, "FORMAT_ERROR", String.join("; ", errorDetails));
                    errorWriter.println(csvLine + ",FORMAT_ERROR," + escapeForCsv(String.join("; ", errorDetails)));
                } else if (isRowEmpty) {
                    handleError(result, i, "EMPTY_ROW", "Complete row is empty");
                    errorWriter.println(buildErrorLine(headers.size(), "EMPTY_ROW", "Row " + (i+1) + " is completely empty"));
                } else if (hasNullValues) {
                    handleError(result, i, "NULL_VALUES", "Contains NULL values");
                    errorWriter.println(csvLine + ",NULL_VALUES,Contains NULL values");
                } else {
                    writer.println(csvLine);
                    result.convertedRows++;
                }
            }

            result.validationPassed = true;
            return result;
        }
    }

    private List<String> findMissingColumns(Row headerRow) {
        List<String> missing = new ArrayList<>();
        for (int i = 0; i < expectedHeaders.length; i++) {
            Cell cell = headerRow.getCell(i);
            if (cell == null || !expectedHeaders[i].equalsIgnoreCase(cell.getStringCellValue())) {
                missing.add(expectedHeaders[i]);
            }
        }
        return missing;
    }

    private List<String> validateHeaders(Row headerRow) {
        List<String> errors = new ArrayList<>();
        for (int i = 0; i < expectedHeaders.length; i++) {
            Cell cell = headerRow.getCell(i);
            if (cell == null || cell.getCellType() != CellType.STRING || 
                !expectedHeaders[i].equalsIgnoreCase(cell.getStringCellValue().trim())) {
                errors.add(String.format("Expected '%s' found '%s' at position %d", 
                    expectedHeaders[i], 
                    cell == null ? "null" : cell.getStringCellValue(), 
                    i+1));
            }
        }
        return errors;
    }

    private List<String> validateColumnFormats(Sheet sheet) {
        List<String> invalidColumns = new ArrayList<>();
        
        // Check first 10 data rows for format validation
        int rowsToCheck = Math.min(10, sheet.getLastRowNum());
        for (int i = 1; i <= rowsToCheck; i++) {
            Row row = sheet.getRow(i);
            if (row == null) continue;
            
            for (int j = 0; j < expectedFormats.length; j++) {
                Cell cell = row.getCell(j);
                if (cell != null && !validateCellFormat(cell, expectedFormats[j])) {
                    String columnName = sheet.getRow(0).getCell(j).getStringCellValue();
                    if (!invalidColumns.contains(columnName)) {
                        invalidColumns.add(columnName);
                    }
                }
            }
        }
        return invalidColumns;
    }

    private boolean validateCellFormat(Cell cell, String expectedFormat) {
        if (cell == null || cell.getCellType() == CellType.BLANK) {
            return true;
        }

        String actualType = getCellTypeName(cell.getCellType());
        
        // Special handling for dates (stored as numeric in Excel)
        if (expectedFormat.equalsIgnoreCase("DATE") && cell.getCellType() == CellType.NUMERIC) {
            return DateUtil.isCellDateFormatted(cell);
        }
        
        return actualType.equalsIgnoreCase(expectedFormat);
    }

    private String getCellTypeName(CellType cellType) {
        if (cellType == null) return "BLANK";
        return cellType.toString();
    }

    private String getCellValueAsString(Cell cell) {
        if (cell == null) {
            return null;
        }

        switch (cell.getCellType()) {
            case STRING:
                return cell.getStringCellValue().trim();
            case NUMERIC:
                if (DateUtil.isCellDateFormatted(cell)) {
                    Date date = cell.getDateCellValue();
                    return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(date);
                } else {
                    double num = cell.getNumericCellValue();
                    if (num == (long) num) {
                        return String.valueOf((long) num);
                    } else {
                        return String.valueOf(num);
                    }
                }
            case BOOLEAN:
                return cell.getBooleanCellValue() ? "TRUE" : "FALSE";
            case FORMULA:
                try {
                    Workbook workbook = cell.getSheet().getWorkbook();
                    FormulaEvaluator evaluator = workbook.getCreationHelper().createFormulaEvaluator();
                    CellValue cellValue = evaluator.evaluate(cell);
                    switch (cellValue.getCellType()) {
                        case NUMERIC:
                            return String.valueOf(cellValue.getNumberValue());
                        case STRING:
                            return cellValue.getStringValue();
                        case BOOLEAN:
                            return cellValue.getBooleanValue() ? "TRUE" : "FALSE";
                        default:
                            return null;
                    }
                } catch (Exception e) {
                    logger.warn("Formula evaluation error at cell {}: {}", cell.getAddress(), e.getMessage());
                    return "#FORMULA_ERROR";
                }
            case BLANK:
                return null;
            case ERROR:
                return "#ERROR";
            default:
                return null;
        }
    }

    private String escapeForCsv(String value) {
        if (value == null || value.isEmpty()) {
            return "";
        }

        String escaped = value.replace("\"", "\"\"");
        if (escaped.contains(",") || escaped.contains("\"") || escaped.contains("\n") || escaped.contains("\r")) {
            escaped = "\"" + escaped + "\"";
        }
        return escaped;
    }

    private void handleError(ConversionResult result, int rowIndex, String errorType, String errorDetails) {
        result.errorRows.add(rowIndex);
        result.errorMessages.add("Row " + (rowIndex+1) + ": " + errorDetails);
        result.errorTypes.merge(errorType, 1, Integer::sum);
        logger.warn("Row {} error: {} - {}", rowIndex+1, errorType, errorDetails);
    }

    private String buildErrorLine(int columns, String errorType, String errorDetails) {
        return String.join(",", Collections.nCopies(columns, "NULL")) + 
               "," + errorType + "," + escapeForCsv(errorDetails);
    }

    private void generateReport(ConversionResult result) throws IOException {
        try (PrintWriter reportWriter = new PrintWriter(new FileWriter(reportFilePath))) {
            // Header
            reportWriter.println("=== XLS to CSV Conversion Report ===");
            reportWriter.println("Generated: " + new Date());
            reportWriter.println("Input File: " + inputFilePath);
            reportWriter.println("Output File: " + outputFilePath);
            reportWriter.println("Status: " + (result.validationPassed ? "SUCCESS" : "FAILED"));

            // Summary
            reportWriter.println("\n[SUMMARY]");
            reportWriter.println("Total rows processed: " + result.totalRows);
            reportWriter.println("Successfully converted: " + result.convertedRows);
            reportWriter.println("Rows with errors: " + result.errorRows.size());
            reportWriter.printf("Success rate: %.2f%%%n", (result.convertedRows * 100.0 / result.totalRows));

            // Validation Issues
            if (!result.missingColumns.isEmpty()) {
                reportWriter.println("\n[MISSING COLUMNS]");
                result.missingColumns.forEach(reportWriter::println);
            }

            if (!result.invalidFormatColumns.isEmpty()) {
                reportWriter.println("\n[INVALID FORMAT COLUMNS]");
                result.invalidFormatColumns.forEach(reportWriter::println);
            }

            // Error Details
            reportWriter.println("\n[ERROR DETAILS]");
            result.errorTypes.forEach((type, count) -> 
                reportWriter.printf("%-15s: %d (%.2f%%)%n", 
                    type, count, (count * 100.0 / result.totalRows)));

            reportWriter.println("\n[ERROR MESSAGES]");
            result.errorMessages.forEach(reportWriter::println);
        }
        logger.info("Generated report file: {}", reportFilePath);
    }
}
```

## **Key Enhancements in This Version**

1. **Comprehensive Validation Pipeline**:
   ```mermaid
   graph TD
   A[Start] --> B{File Exists?}
   B -->|Yes| C{File Not Empty?}
   C -->|Yes| D{Column Count Match?}
   D -->|Yes| E{Header Names Match?}
   E -->|Yes| F{Column Formats Match?}
   F -->|Yes| G{Contains Data Rows?}
   G -->|Yes| H[Convert Data]
   ```

2. **Detailed Error Reporting**:
   - Tracks missing columns by name
   - Identifies invalid format columns
   - Provides exact header mismatch positions
   - Logs all validation failures

3. **Enhanced Logging**:
   - INFO level for major milestones
   - WARN level for validation issues
   - ERROR level for critical failures
   - DEBUG level for detailed processing

4. **Validation Improvements**:
   - Checks first 10 data rows for format validation
   - Provides exact expected vs actual comparisons
   - Tracks all validation issues separately

5. **Complete Reporting**:
   - Includes all validation issues in report
   - Shows success rate
   - Lists all error messages
   - Tracks missing/invalid columns

## **Configuration Properties Example**

```properties
# File paths
input.file.path=input/data.xls
output.file.path=output/converted.csv
error.file.path=output/errors.csv
report.file.path=output/conversion_report.txt

# Validation parameters
expected.column.count=5
expected.column.headers=ID,Name,Age,JoinDate,Active
expected.column.formats=STRING,STRING,NUMERIC,DATE,BOOLEAN
```

## **Usage Example**

```java
@Autowired
private JobLauncher jobLauncher;

@Autowired
private Job xlsToCsvConversionJob;

public void runConversion() {
    try {
        JobParameters params = new JobParametersBuilder()
            .addLong("time", System.currentTimeMillis())
            .toJobParameters();
        
        JobExecution execution = jobLauncher.run(xlsToCsvConversionJob, params);
        
        // Get conversion results
        ConversionResult result = (ConversionResult) execution.getExecutionContext()
            .get("conversionReport");
        
        logger.info("Conversion completed with {} errors", result.errorRows.size());
    } catch (Exception e) {
        logger.error("Job failed", e);
    }
}
```

This implementation provides a complete, production-ready solution with all requested validations, detailed error reporting, and comprehensive logging throughout the conversion process.
