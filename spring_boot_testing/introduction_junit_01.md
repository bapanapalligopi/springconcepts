what is junit?
=====================================
- junit is testing framework for java
- it is used to write and run repeatable tests
- junit helps verify that individuals units of code work as expected.

JUnit is a widely used testing framework in Java designed to perform unit testing. It allows developers to write test cases to verify that individual units of code, such as methods or classes, behave as expected.

how junit works?
=====================================
=> first we will write the method for testing the code and then we will annotate the method with @Test annotation.
=> junit provides the assertions to check the if the results are as expected.
=> junit platform has the test runners , test runners has the capable of execute the tests and report the results.
=> junit can be integrsted with IDE and build tools fot easy execution.

java code => class under the test
unit test => using junit framework
test runners=> junit runners

key features of Junit
============================
annotations make test management easy
assertions are used to check the expected results
test runners execute your tests ans report the results
integration wuth build tool like maven and gradle

junit basic concepts
=================================
system under the test or class under test => the class or system we want to test  
method under the test => the method or function we want to test

The "method under test" refers to the actual method from the class being tested. It is the method whose functionality is being verified using assertions.

test class=> normal class containd lot og test methods and every method is  annotated with the @Test
test method is a method in test class contains small piece of code that test the perticular method in test class
assertions are use d to check the excepted resuts in methods of test class  and some of most famous asserions are assertEquals,True,FalseThrows

tets runner is responsible for executing the tet methods in a test class and report the results.
The test runner is responsible for running the test methods, executing them, and reporting whether the tests passed or failed.

setting up environment for junit
===================================
create a maven project
add the dependencies


<!-- https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-api -->
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.11.3</version>
    <scope>test</scope>
</dependency>


@Test annotation
============================
The @Test annotation is used to identify a method as a test method. When a method is annotated with @Test, JUnit recognizes it as a test to be executed.
marks the method as test method
junit automatically ruins all methods annotatted with @Test with in that test class or package
from org.junit.jupiter.api package
public ,protected,default visibility, private in junit5
test class also can be public ,default.
In JUnit, test methods annotated with @Test must have a void return type, meaning they cannot return any value.
If a test fails in JUnit 5, an AssertionError is thrown when the actual output doesn't match the expected output.
In JUnit 5, methods annotated with @Test can have public, protected, or package-private (default) visibility. They cannot be private, as JUnit needs access to these methods to run them as tests.
