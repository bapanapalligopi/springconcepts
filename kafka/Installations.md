To install **Apache Kafka** on **Windows**, follow these step-by-step instructions. Apache Kafka relies on **Apache ZooKeeper** for managing cluster state, so you need to have both Kafka and ZooKeeper running on your machine.

### Prerequisites:
- **Java**: Kafka is written in Java, so you need to have **Java** installed on your system.
  - Kafka requires **Java 8 or newer**.

### Step 1: Install Java
1. **Download and Install Java**:
   - Go to the official [Java download page](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html) or download the latest version of OpenJDK from [AdoptOpenJDK](https://adoptopenjdk.net/).
   - Follow the installation instructions for your system.

2. **Set up JAVA_HOME**:
   - After installing Java, you need to set the `JAVA_HOME` environment variable:
     1. Open **Control Panel** -> **System** -> **Advanced system settings**.
     2. Click on **Environment Variables**.
     3. Under **System variables**, click **New** and set:
        - **Variable Name**: `JAVA_HOME`
        - **Variable Value**: Path to your Java installation (e.g., `C:\Program Files\AdoptOpenJDK\jdk-11.0.10.9-hotspot`)
     4. Add the Java `bin` directory to your `PATH` variable, so that the `java` command works globally.

### Step 2: Install ZooKeeper
Kafka uses ZooKeeper for distributed coordination. Luckily, Kafka includes ZooKeeper as part of the download, so you don't need to install ZooKeeper separately for a basic setup.

### Step 3: Download Apache Kafka
1. Go to the official **Kafka download page**: [Kafka Download](https://kafka.apache.org/downloads).
2. Download the **binary** package (e.g., `kafka_2.13-2.8.0.tgz`).
3. Extract the downloaded `.tgz` file to a folder (e.g., `C:\kafka`).

### Step 4: Configure Kafka
1. **Navigate to the Kafka folder**: 
   - Open a **Command Prompt** and navigate to the Kafka folder, for example:
     ```
     cd C:\kafka
     ```

2. **Modify the Kafka configuration** (optional):
   - By default, Kafka uses ZooKeeper on `localhost:2181`, which should work for a local installation.
   - Kafka's configuration file is located at `config\server.properties`. You can open this file in a text editor to modify settings such as `listeners` or `log.dirs`, but for most basic setups, the default values are fine.

### Step 5: Start ZooKeeper
1. In the same **Command Prompt**, navigate to the Kafka folder and run the following command to start ZooKeeper:
   ```bash
   .\bin\windows\zookeeper-server-start.bat .\config\zookeeper.properties
   ```

   This will start the ZooKeeper server, and you should see log output indicating that ZooKeeper is running.

### Step 6: Start Kafka Broker
1. Open another **Command Prompt** window (you need two separate windows to run Kafka and ZooKeeper simultaneously).
2. In the second Command Prompt, navigate to the Kafka folder and run the following command to start the Kafka broker:
   ```bash
   .\bin\windows\kafka-server-start.bat .\config\server.properties
   ```

   This will start the Kafka broker. You should see log output indicating that Kafka is successfully running.

### Step 7: Create a Kafka Topic (Optional)
After starting Kafka, you can create a topic to test it out. Run the following command in another Command Prompt window:
```bash
.\bin\windows\kafka-topics.bat --create --topic test --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
```

- `--topic test`: Creates a topic named `test`.
- `--bootstrap-server localhost:9092`: The Kafka broker to connect to.
- `--partitions 1`: Number of partitions for the topic.
- `--replication-factor 1`: Replication factor (you can set it to 1 for single-node setups).

### Step 8: Test Kafka Producer and Consumer

1. **Start a Kafka Producer**:
   To send messages to the `test` topic, run the following command:
   ```bash
   .\bin\windows\kafka-console-producer.bat --broker-list localhost:9092 --topic test
   ```

   This will open a prompt where you can type messages. Every message you type will be sent to the `test` topic.

2. **Start a Kafka Consumer**:
   To consume the messages from the `test` topic, run the following command in another Command Prompt:
   ```bash
   .\bin\windows\kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test --from-beginning
   ```

   This will display the messages sent to the `test` topic in the producer terminal.

---

### Step 9: Stop Kafka and ZooKeeper
1. To stop Kafka, press **Ctrl + C** in the Kafka Command Prompt window.
2. To stop ZooKeeper, press **Ctrl + C** in the ZooKeeper Command Prompt window.

---

### Troubleshooting Tips:

- **Check Java Version**: If Kafka fails to start, make sure your `JAVA_HOME` is set correctly and that you're using a supported version of Java (Java 8 or newer).
- **Ports**: Kafka uses port `9092` by default. If you have other services using that port, you may need to modify the `server.properties` file to change the port.
- **Zookeeper Error**: If ZooKeeper doesn't start, check the `zookeeper.properties` file for configuration issues or port conflicts.

---

### Conclusion

By following the above steps, you should have **Apache Kafka** up and running on your Windows machine. Kafka can now be used to send and receive messages in real time, and you can start building real-time data streaming applications.

files for linux and windows operating system in kafka looks like below image

![image](https://github.com/user-attachments/assets/ec238d2c-5169-4cb4-a555-7f7b20374c94)
