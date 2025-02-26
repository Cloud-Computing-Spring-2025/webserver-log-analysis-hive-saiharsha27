## **Web Log Analysis Using Hive & Hadoop**

### **Project Overview**
This project analyzes web server logs using Apache Hive on a Hadoop ecosystem. It extracts insights like total requests, status codes, most visited pages, user agents, suspicious IPs, and traffic trends. The goal is to process large-scale logs efficiently and gain actionable insights.

---

### **Implementation Approach**
Each analysis task is performed using Hive queries on structured log data stored in HDFS.

- **Total Web Requests**:  
  ```sql
  SELECT COUNT(*) AS total_requests FROM web_logs;
  ```

- **Status Code Analysis**:  
  ```sql
  SELECT status_code, COUNT(*) AS count
  FROM web_logs
  GROUP BY status_code;
  ```

- **Most Visited Pages**:  
  ```sql
  SELECT page, COUNT(*) AS visits
  FROM web_logs
  GROUP BY page
  ORDER BY visits DESC
  LIMIT 10;
  ```

- **Traffic Source Analysis**:  
  ```sql
  SELECT user_agent, COUNT(*) AS count
  FROM web_logs
  GROUP BY user_agent
  ORDER BY count DESC;
  ```

- **Suspicious IP Addresses (Frequent Errors)**:  
  ```sql
  SELECT ip_address, COUNT(*) AS failed_requests
  FROM web_logs
  WHERE status_code IN (400, 403, 404, 500)
  GROUP BY ip_address
  HAVING COUNT(*) > 3;
  ```

- **Traffic Trend Over Time**:  
  ```sql
  SELECT request_time, COUNT(*) AS requests
  FROM web_logs
  GROUP BY request_time
  ORDER BY request_time;
  ```

---

### **Execution Steps**
1. **Start Docker and Hadoop Services**  
   ```bash
   docker compose up -d
   ```

2. **Load the CSV data into HDFS and then into the Hive table**

2. **Start Hue: which is a web-interface to access Hive. Can be accessed at port:8888**

3. **Run Hive Queries in the Hue editor**

3. **Export Query Results from HDFS**
   ```bash
   hdfs dfs -getmerge /user/hue/output/total_requests total_requests.txt
   hdfs dfs -getmerge /user/hue/output/status_codes status_codes.txt
   hdfs dfs -getmerge /user/hue/output/most_visited_pages most_visited_pages.txt
   hdfs dfs -getmerge /user/hue/output/user_agents user_agents.txt
   hdfs dfs -getmerge /user/hue/output/suspicious_ips suspicious_ips.txt
   hdfs dfs -getmerge /user/hue/output/traffic_trends traffic_trends.txt
   ```

5. **Copy Results to Local Machine**
   ```bash
   docker cp resourcemanager:/total_requests.txt output/
   docker cp resourcemanager:/status_codes.txt output/
   docker cp resourcemanager:/most_visited_pages.txt output/
   docker cp resourcemanager:/user_agents.txt output/
   docker cp resourcemanager:/suspicious_ips.txt output/
   docker cp resourcemanager:/traffic_trends.txt output/
   ```

---

### **Challenges Faced & Resolutions**
1. **HDFS Output Directory Not Found**  
   - **Issue**: Error when fetching results from HDFS.
   - **Solution**: Ensured Hive writes output to `/user/hue/output/` and verified with `hdfs dfs -ls /user/hue/output/`.

2. **Hive Query Execution Timeout**  
   - **Issue**: Queries taking too long on large logs.
   - **Solution**: Optimized queries using partitions and indexes.

3. **Docker Directory Missing for Output**  
   - **Issue**: Errors like `/opt/hadoop-3.2.1/share/hadoop/mapreduce/': No such file or directory`
   - **Solution**: Used the correct HDFS path and verified files with `hdfs dfs -ls`.

---

### **Sample Input & Output**
#### **Sample Log Entry**
```
192.168.1.10 - - [2024-02-01 10:15] "GET /home" 200 "Mozilla/5.0"
192.168.1.11 - - [2024-02-01 10:16] "GET /products" 404 "Chrome/90.0"
192.168.1.12 - - [2024-02-01 10:17] "POST /checkout" 500 "Safari/13.1"
```

#### **Expected Output**
```
Total Web Requests:
Total Requests: 100

Status Code Analysis:
200: 80
404: 10
500: 10

Most Visited Pages:
/home: 50
/products: 30
/checkout: 20

Traffic Source Analysis:
Mozilla/5.0: 60
Chrome/90.0: 30
Safari/13.1: 10

Suspicious IP Addresses:
192.168.1.10: 5 failed requests
192.168.1.15: 4 failed requests

Traffic Trend Over Time:
2024-02-01 10:15: 5 requests
2024-02-01 10:16: 7 requests
```

