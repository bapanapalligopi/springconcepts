
### **EJB (Enterprise JavaBeans)**

#### **Description**:
The `ejb` packaging type is used specifically for **Enterprise JavaBeans (EJB)** modules in a Java EE (now Jakarta EE) application. EJB is a specification that provides a platform for developing and deploying enterprise-level applications that encapsulate business logic. These beans can be categorized into:

- **Session Beans**:
  - **Stateless Session Beans**: These do not maintain any client-specific state between method invocations.
  - **Stateful Session Beans**: These maintain client-specific state during a conversation.
  
- **Message-Driven Beans (MDBs)**: Used for handling asynchronous messaging via **JMS (Java Message Service)**.

The `ejb` packaging type in Maven is used when you're building an EJB module that contains business logic, which will be deployed on a Java EE application server like **WildFly**, **GlassFish**, **WebLogic**, or **JBoss**.

#### **Use Case**:
- When the core business logic is encapsulated within **EJBs**, and you want to deploy just the EJB component without a web layer (WAR), or when the EJBs are part of a larger EAR (Enterprise Archive) that also includes web and other modules.
- **Session Beans**: For managing business operations.
- **Message-Driven Beans**: For asynchronous message processing (e.g., consuming JMS messages).

### **Steps for Creating an EJB Module**:

1. **Set up a Maven project with `ejb` packaging**.
2. **Write the business logic** (Session Beans or Message-Driven Beans).
3. **Compile and package the project** into a `.ejb` file (or `.jar` in an EAR).
4. **Deploy the packaged EJB module** to a Java EE server.

---

### **Example: EJB Project with Maven**

Let's go step by step to see how you can create a Maven project for EJB.

#### 1. **Create a Maven Project (`pom.xml`)**

For an EJB project, the Maven `pom.xml` file needs to specify the `ejb` packaging type and necessary dependencies for EJBs.

##### **Example `pom.xml`**:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>ejb-example</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>ejb</packaging> <!-- EJB packaging type -->

    <dependencies>
        <!-- EJB API dependency -->
        <dependency>
            <groupId>javax.ejb</groupId>
            <artifactId>javax.ejb-api</artifactId>
            <version>3.2</version> <!-- EJB version -->
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <!-- Maven EAR plugin (if you're packaging EJBs into an EAR later) -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-ear-plugin</artifactId>
                <version>3.1.0</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>ear</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

#### **Explanation of `pom.xml`**:
- The `<packaging>` element is set to `ejb`, indicating that the Maven project is intended to build an EJB module.
- The dependency on `javax.ejb-api` version `3.2` is included, which provides the necessary interfaces and annotations for developing EJBs.
  
#### 2. **Write the EJB Code**

Now, let's create an example **Stateless Session Bean** and deploy it in the EJB module.

##### **Example of a Stateless Session Bean** (`HelloWorldEJB.java`):

```java
package com.example.ejb;

import javax.ejb.Stateless;

@Stateless
public class HelloWorldEJB implements HelloWorldEJBLocal {

    @Override
    public String sayHello() {
        return "Hello from EJB!";
    }
}
```

In this code:
- `@Stateless`: This annotation indicates that the `HelloWorldEJB` is a **Stateless Session Bean**.
- `HelloWorldEJBLocal` is a local interface that defines the business method (`sayHello`).

##### **Local Interface** (`HelloWorldEJBLocal.java`):

```java
package com.example.ejb;

import javax.ejb.Local;

@Local
public interface HelloWorldEJBLocal {
    String sayHello();
}
```

The `@Local` annotation marks the interface as a local business interface, meaning that the EJB can be accessed within the same JVM.

#### 3. **Build the EJB Module**

After setting up the `pom.xml` and writing the necessary Java code, you can use Maven to compile and package the EJB module.

- To **build** the project and generate the `.ejb` (or `.jar` file) from the EJB module, run:

```bash
mvn clean install
```

This command will:
1. Clean the previous build artifacts (if any).
2. Compile the source code and package the EJB module into a `.jar` (since `ejb` modules are generally packaged as `.jar` files in the absence of an EAR).
3. Install the packaged `.ejb` artifact into your local Maven repository (usually in `~/.m2/repository`).

After running the command, the output will be located in the `target` directory, typically as a `.jar` file.

Example:
```bash
target/ejb-example-1.0-SNAPSHOT.jar
```

#### 4. **Deploy the EJB Module to an Application Server**

To deploy the generated `.jar` file containing the EJB to a Java EE server, such as **WildFly** or **JBoss**:

- Copy the `.jar` file into the `deployments/` directory of your application server.
  For example, if you're using WildFly:
  
```bash
cp target/ejb-example-1.0-SNAPSHOT.jar $WILDFLY_HOME/standalone/deployments/
```

Alternatively, you can use the **WildFly Management CLI** to deploy the EJB module:

```bash
./jboss-cli.sh --connect --command="deploy target/ejb-example-1.0-SNAPSHOT.jar"
```

#### 5. **Accessing the EJB in Another Module or Application**

Once the EJB is deployed, you can access it from other modules in your Java EE application. For instance, if you have a **Servlet** or **JAX-RS** resource in a `war` (Web Application), you can inject and use the `HelloWorldEJB`:

##### Example of EJB injection in a Servlet:

```java
package com.example.web;

import com.example.ejb.HelloWorldEJBLocal;

import javax.ejb.EJB;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class HelloWorldServlet extends HttpServlet {

    @EJB
    private HelloWorldEJBLocal helloWorldEJB;

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.getWriter().println(helloWorldEJB.sayHello());
    }
}
```

Here, the `@EJB` annotation injects the `HelloWorldEJBLocal` bean into the `HelloWorldServlet`. When a request is made to the servlet, it will use the EJB to return the message `"Hello from EJB!"`.

---

### **Conclusion**

1. **EJB (Enterprise JavaBeans)** is used to encapsulate business logic, such as session beans (stateful or stateless) or message-driven beans (MDBs), in enterprise applications.
2. **Maven EJB Module**: The `ejb` packaging type is used when you're developing and packaging an EJB module. The `pom.xml` file needs to specify the `ejb` packaging type and necessary dependencies (like `javax.ejb-api`).
3. **EJB Deployment**: Once the EJB module is built, it can be deployed to a Java EE application server (e.g., WildFly or JBoss). The EJBs can then be accessed from other modules, such as web modules (WAR).

By following the steps above, you can build a Java EE EJB project, package it with Maven, and deploy it to an application server.
