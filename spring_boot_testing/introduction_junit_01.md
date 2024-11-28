
### What is JUnit?
JUnit is a testing framework for Java that is used to write and run repeatable tests. It is primarily used to verify that individual units of code (such as methods or classes) work as expected.

- **JUnit is a framework**: It provides a set of tools to write and run tests for Java applications.
- **Used for unit testing**: It helps in testing individual parts of the code (methods or functions).
- **Repeatable tests**: Tests can be run multiple times to ensure that changes don't break existing functionality.

### How JUnit Works?
- **Write the method**: The first step is to write the method that you want to test. This method will be placed under a test class.
- **Annotate with `@Test`**: After writing the method, we annotate the testing method with the `@Test` annotation. This lets JUnit know that this method should be treated as a test.
- **Assertions**: JUnit provides assertion methods like `assertEquals()`, `assertTrue()`, `assertFalse()`, etc., to check whether the actual results match the expected results.
- **Test Runners**: JUnit test runners are responsible for executing the tests and reporting the results. JUnit supports integration with IDEs like Eclipse, IntelliJ IDEA, and build tools like Maven and Gradle.

In JUnit:
- **Java code**: Refers to the class you are testing.
- **Unit test**: The test cases you write using JUnit framework to test the Java code.
- **Test runners**: Tools in JUnit that execute the test cases and report the outcome.

### Key Features of JUnit
- **Annotations**: JUnit uses annotations like `@Test`, `@Before`, `@After` to make test management easier.
- **Assertions**: Assertions are used to check whether the results are as expected.
- **Test Runners**: Test runners are responsible for executing the tests and reporting results.
- **Integration**: JUnit integrates well with build tools like Maven and Gradle.

### JUnit Basic Concepts
1. **System Under Test (SUT)**: The class or system you want to test.
2. **Method Under Test**: The method or function in the class that you are testing.

The "method under test" refers to the actual method that you want to verify using assertions. 

- **Test Class**: A normal Java class that contains multiple test methods. Every method in this class is annotated with `@Test`.
- **Test Method**: A method in the test class that contains code to test a particular method or function in the class being tested.
- **Assertions**: Assertions are used in test methods to check whether the results match the expected values. Some of the most commonly used assertions are:
  - `assertEquals()`
  - `assertTrue()`
  - `assertFalse()`
  - `assertThrows()`
  
- **Test Runner**: The test runner is responsible for executing the test methods in the test class and reporting the results. Test runners can be configured to run tests in specific environments and report detailed test results.

### Setting Up Environment for JUnit
1. **Create a Maven Project**: If you're using Maven, you need to add JUnit dependencies to your `pom.xml` file to enable JUnit in your project.
2. **Add Dependencies**: Below is an example of how to add the JUnit 5 dependency to your Maven project:

```xml
<!-- https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-api -->
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.11.3</version>
    <scope>test</scope>
</dependency>
```

This dependency allows you to use JUnit in your project, which is required for writing and running tests.

### `@Test` Annotation
The `@Test` annotation is used to mark a method as a test case in JUnit. When a method is annotated with `@Test`, JUnit recognizes it as a test method that should be executed. 

- **Marks the method as a test**: By annotating a method with `@Test`, you are indicating that this method is a test case.
- **JUnit automatically runs methods annotated with `@Test`**: Once a method is annotated, the JUnit test runner will automatically execute it when you run the tests.
- **Visibility**: In JUnit 5, test methods annotated with `@Test` can be public, protected, or package-private (default), but they cannot be private.
- **Return Type**: Test methods should have a `void` return type. They do not return any values.
- **Failure**: If the test fails, an `AssertionError` is thrown, and JUnit reports that the actual result does not match the expected result.

### Example Code with JUnit

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;

public class CalculatorTest {

    @Test
    public void testAdd() {
        Calculator calculator = new Calculator();
        int result = calculator.add(5, 3);
        assertEquals(8, result, "Addition result should be 8");
    }
}
```

In this example:
- We have a method `testAdd()` that tests the `add()` method of a `Calculator` class.
- The `assertEquals()` assertion is used to verify that the result of the `add(5, 3)` method call is equal to `8`.

### Sample Image:
Here is an example image demonstrating Spring Boot testing with JUnit:

![Spring Boot Testing Example](https://github.com/bapanapalligopi/springconcepts/blob/main/spring_boot_testing/images/junitrecentimage.png)

---

With this, you've learned about JUnit's basic concepts, how to set it up, and how to write simple test cases to verify the functionality of your code.
Here is a more polished and well-structured version of the content, ensuring clarity and correctness:

---

### **JUnit Assertion Methods**

The `Assertions` class in JUnit provides static methods for verifying test conditions. These methods play a vital role in validating the expected outcomes of test cases.

---

#### **1. `assertTrue` Method**
- **Purpose:** Validates that the condition supplied is `true`.
- **Behavior:** If the condition is `true`, the test case passes. Otherwise, it fails.
- **Overloaded Methods:** There are six overloaded variants:
  1. `assertTrue(boolean actual)`  
     Validates a boolean condition directly.
  2. `assertTrue(Supplier<Boolean> actualSupplier)`  
     Executes lazily using Java 8 functional programming. The condition is evaluated only when assertions are executed.
  3. `assertTrue(boolean actual, String message)`  
     Provides a custom failure message if the condition is false.
  4. `assertTrue(Supplier<Boolean> actualSupplier, String message)`  
     Lazy evaluation of the boolean supplier, with a custom failure message.
  5. `assertTrue(boolean actual, Supplier<String> messageSupplier)`  
     Executes the custom message logic only if the condition is false.
  6. `assertTrue(Supplier<Boolean> actualSupplier, Supplier<String> messageSupplier)`  
     Both the boolean supplier and custom message supplier are evaluated lazily.

**Example:**  
```java
assertTrue(condition, "Condition must be true");
assertTrue(() -> someLazyCondition(), () -> "Lazy evaluation failed message");
```

---

#### **2. `assertFalse` Method**
- **Purpose:** Opposite of `assertTrue`, it validates that the condition is `false`.
- **Behavior:** If the condition is `false`, the test case passes; otherwise, it fails.
- **Overloaded Methods:** The same six variants are available, mirroring `assertTrue`.

---

#### **3. `assertNull` Method**
- **Purpose:** Checks whether an object is `null`.
- **Behavior:** If the object is `null`, the test case passes; otherwise, it fails.
- **Overloaded Methods:**  
  1. `assertNull(Object actual)`  
  2. `assertNull(Object actual, String message)`  
  3. `assertNull(Object actual, Supplier<String> messageSupplier)`  

**Example:**  
```java
assertNull(someObject, "Object should be null");
```

---

#### **4. `assertNotNull` Method**
- **Purpose:** Checks whether an object is not `null`.
- **Behavior:** If the object is not `null`, the test case passes; otherwise, it fails.
- **Overloaded Methods:**  
  1. `assertNotNull(Object actual)`  
  2. `assertNotNull(Object actual, String message)`  
  3. `assertNotNull(Object actual, Supplier<String> messageSupplier)`  

**Example:**  
```java
assertNotNull(someObject, "Object should not be null");
```

---

### **Code Examples**

#### **`Student` Model**
```java
package com.junitpractice.demo.models;

public class Student {

    private int id;
    private String name;

    // Getters and setters
    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    // Constructor
    public Student(int id, String name) {
        this.id = id;
        this.name = name;
    }
}
```

---

#### **`StudentService` Class**
```java
package com.junitpractice.demo.service;

import java.util.ArrayList;
import java.util.List;

import com.junitpractice.demo.models.Student;

public class StudentService {

    private List<Student> students = new ArrayList<>();

    public List<Student> getStudents() {
        return this.students;
    }

    public void addStudent(Student student) {
        students.add(student);
    }

    public Student getStudentById(int id) {
        return students.stream()
                .filter(student -> student.getId() == id)
                .findFirst()
                .orElse(null);
    }
}
```

---

#### **`StudentServiceTest` Class**
```java
package com.junitpractice.demo.service;

import static org.junit.jupiter.api.Assertions.*;

import org.junit.jupiter.api.Test;

import com.junitpractice.demo.models.Student;

class StudentServiceTest {

    @Test
    public void getStudentsTestUsingAssertionTrue() {
        StudentService service = new StudentService();

        Boolean actualResults = service.getStudents().isEmpty();

        assertTrue(actualResults, "The students list should be empty");
    }

    @Test
    public void getStudentsTestUsingAssertionTrueNot() {
        StudentService service = new StudentService();
        Boolean actualResults = service.getStudents().isEmpty();

        assertTrue(() -> actualResults, () -> "The students list is empty");
    }

    @Test
    public void getStudentsListFalse() {
        StudentService service = new StudentService();
        Boolean actualResults = service.getStudents().isEmpty();

        assertFalse(() -> actualResults, () -> "The students list is not empty");
    }

    @Test
    public void addStudentTestNullCheck() {
        StudentService service = new StudentService();
        Student student = null;

        assertNull(student, "The student object is null, so it cannot be added");
    }

    @Test
    public void addStudentTestNullCheckFalse() {
        StudentService service = new StudentService();
        Student student = new Student(1, "Student Name");

        service.addStudent(student);

        assertNotNull(student, "A student object was found");
    }

    @Test
    public void getStudentByIdTest() {
        StudentService service = new StudentService();
        Student student = new Student(1, "Gopi");
        service.addStudent(student);

        Student actualStudent = service.getStudentById(1);

        assertNotNull(actualStudent, "The student should not be null when fetched by ID");
    }

    @Test
    public void getStudentByIdTestNegative() {
        StudentService service = new StudentService();

        Student actualStudent = service.getStudentById(1);

        assertNull(actualStudent, "No student should be found with this ID");
    }
}
```

--- 

