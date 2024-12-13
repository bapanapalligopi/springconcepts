Here's a summary of all the commands you've run and need to run for setting up and verifying your Kafka and ZooKeeper environment on Windows:

### **1. Start ZooKeeper**
Start ZooKeeper with the following command (in `C:\kafka\bin\windows` directory):
```cmd
zookeeper-server-start.bat config\zookeeper.properties
```

### **2. Test ZooKeeper Connection**
Verify ZooKeeper is running by using the ZooKeeper shell (in `C:\kafka\bin\windows` directory):
```cmd
zookeeper-shell.bat localhost:2181
```
Once inside the shell, list the root directory:
```bash
ls /
```
You should see something like:
```bash
[zookeeper]
```

### **3. Start Kafka**
Start Kafka (in `C:\kafka\bin\windows` directory):
```cmd
kafka-server-start.bat config\server.properties
```

### **4. Verify Kafka Logs**
Kafka logs should indicate that it's successfully started. Check logs for messages like:
```plaintext
[KafkaServer id=0] started
```
Kafka is now listening on port `9092` for connections.

### **5. Create a Kafka Topic**
To create a new Kafka topic (e.g., `test-topic`), run:
```cmd
kafka-topics.bat --create --topic test-topic --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
```

### **6. List Kafka Topics**
To verify that the topic was created, list the topics:
```cmd
kafka-topics.bat --list --bootstrap-server localhost:9092
```

### **7. Produce Messages to the Topic**
Start producing messages to the topic (e.g., `test-topic`):
```cmd
kafka-console-producer.bat --topic test-topic --bootstrap-server localhost:9092
```
You can type messages and hit Enter to send them.

### **8. Consume Messages from the Topic**
Start consuming messages from the topic:
```cmd
kafka-console-consumer.bat --topic test-topic --bootstrap-server localhost:9092 --from-beginning
```

---

### **Additional Commands (Optional)**

#### **9. Verify ZooKeeper Connection from Kafka Logs**
If you're experiencing connectivity issues between Kafka and ZooKeeper, you can check the following command from the Kafka logs:
```plaintext
ls /brokers/topics
```
This command will show the topics in ZooKeeper.

#### **10. Clean Up ZooKeeper State (if necessary)**
If ZooKeeper data is corrupted, you may need to clean up by stopping ZooKeeper and deleting its data:
1. Stop ZooKeeper.
2. Delete the contents of the `dataDir` folder (default is `C:/kafka/data/zookeeper`).
3. Restart ZooKeeper.

---

### **Troubleshooting Commands:**

If you encounter issues, here are some helpful commands for debugging:

#### **Check for Port Conflicts**:
To ensure no other services are using Kafka's or ZooKeeper's ports:
```cmd
netstat -ano | findstr 2181  # For ZooKeeper port
netstat -ano | findstr 9092  # For Kafka port
```
If there are conflicting processes, use:
```cmd
taskkill /PID <PID> /F
```

---

This is the full list of commands you've run or may need to run to get Kafka and ZooKeeper working together on your Windows machine. If you run into any more issues or need further clarification, feel free to ask!
