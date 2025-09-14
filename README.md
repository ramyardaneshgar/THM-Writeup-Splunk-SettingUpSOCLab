# Splunk-SettingUpSOCLab
Splunk: Setting Up a SOC - Installing, configuring, and integrating log sources on Linux and Windows environments for centralized monitoring.

**Developed by:** Ramyar Daneshgar  

---

### **Overview**  
This lab focused on setting up and configuring **Splunk** on both Linux and Windows machines, enabling data ingestion through Universal Forwarders, and integrating log sources such as system logs, event logs, and web server logs. The process required working with **CLI tools**, **Splunk web interface**, and interacting with log files directly for real-world SIEM deployments. Below, I document my technical approach, tools, commands, and rationale for each step.

---

## **1. Setting up Splunk on Linux**  

### **Task:** Install and configure Splunk Enterprise on a Linux Server.  

#### **Approach:**  
1. **Unpacking Splunk Installer**:  
   Splunk was already downloaded in the `~/Downloads/splunk` directory as `splunk_installer.tgz`. I started by uncompressing the installer:  
   ```bash
   tar xvzf splunk_installer.tgz
   ```  
   - **Why?**: This extracts the necessary files for Splunk installation.

2. **Moving Splunk to `/opt`**:  
   I moved the extracted Splunk folder to `/opt` for standardization:  
   ```bash
   mv splunk /opt/
   cd /opt/splunk/bin
   ```  

3. **Starting Splunk**:  
   I initiated the Splunk service and accepted the license agreement:  
   ```bash
   ./splunk start --accept-license
   ```  
   During this step, I created the **admin credentials** (username and password).  

4. **Verification**:  
   Once installed, the web interface was accessible via `http://MACHINE_IP:8000`.  

#### **Rationale:**  
Splunk Enterprise is installed to centralize log collection and analysis. Deploying it on Linux ensures robust performance for ingestion and processing.

---

## **2. Interacting with Splunk CLI**  

### **Task:** Use CLI commands to interact with Splunk.  

#### **Commands Used and Why**:  
1. **Start/Stop/Restart Splunk**:  
   - `./splunk start`: Starts the Splunk service.  
   - `./splunk stop`: Stops all running processes.  
   - `./splunk restart`: Useful after configuration changes.  

2. **Checking Status**:  
   ```bash
   ./splunk status
   ```  
   - **Why?**: Ensures Splunk services are running without errors.

3. **Searching Logs**:  
   I searched for specific terms like "coffely" in the indexed logs:  
   ```bash
   ./splunk search coffely
   ```  
   - **Why?**: Validates data ingestion and log retrieval using CLI.

---

## **3. Configuring Universal Forwarder on Linux**  

### **Task:** Install and configure Splunk Universal Forwarder to send system logs to the Splunk indexer.  

#### **Approach:**  
1. **Install Forwarder**:  
   I uncompressed the forwarder package and moved it to `/opt`:  
   ```bash
   tar xvzf splunkforwarder.tgz  
   mv splunkforwarder /opt/  
   cd /opt/splunkforwarder/bin
   ./splunk start --accept-license
   ```  
   I created admin credentials and set a custom management port (8090) since 8089 was occupied.  

2. **Adding Forward Server**:  
   Configured the forwarder to send logs to the Splunk indexer:  
   ```bash
   ./splunk add forward-server MACHINE_IP:9997
   ```  
   - **Why?**: Port `9997` is the default for Splunk receiving forwarded logs.  

3. **Monitoring System Logs**:  
   I configured the forwarder to monitor `/var/log/syslog` and send it to a custom index (`Linux_host`):  
   ```bash
   ./splunk add monitor /var/log/syslog -index Linux_host
   ```  

4. **Validation**:  
   I checked the `inputs.conf` file to ensure the monitor configuration was correct:  
   ```bash
   cat /opt/splunkforwarder/etc/apps/search/local/inputs.conf
   ```  

5. **Generating Test Logs**:  
   To validate, I used the `logger` command:  
   ```bash
   logger "coffely-has-the-best-coffee-in-town"
   ```  

#### **Rationale:**  
Universal Forwarder is lightweight and ideal for sending logs from endpoints to the Splunk indexer. Monitoring critical files like `syslog` ensures visibility into system activities.  

---

## **4. Installing Splunk on Windows**  

### **Task:** Install Splunk Enterprise and configure Universal Forwarder to ingest Windows Event Logs.  

#### **Approach:**  
1. **Installation**:  
   I ran the installer from the `Downloads` directory and installed Splunk to the default path:  
   ```
   C:\Program Files\Splunk
   ```  

2. **Configure Forwarder**:  
   - Installed Splunk Universal Forwarder.  
   - Configured Splunk to receive logs on port `9997`.  
   - Added the Windows host as a data source for **Local Event Logs**.  

3. **Monitoring Logs**:  
   I selected key Windows Event Logs, such as:  
   - **4624**: Successful logins.  

4. **Verification**:  
   Searched for logs with `EventCode=4624` to confirm ingestion:  
   ```spl
   index=windows_logs EventCode=4624
   ```  

#### **Rationale:**  
Windows Event Logs are critical for detecting security events like user authentication and system changes. The forwarder efficiently streams these logs to Splunk for analysis.  

---

## **5. Ingesting Coffely Web Logs**  

### **Task:** Integrate web server logs hosted on Windows IIS into Splunk.  

#### **Approach:**  
1. **Log Location**:  
   The logs were located in:  
   ```
   C:\inetpub\logs\LogFiles\W3SVC*
   ```  

2. **Source Configuration**:  
   Configured the Splunk forwarder to monitor this directory.  

3. **Setting Source Type**:  
   Selected **IIS Server** as the source type and created a new index for web logs.  

4. **Validation**:  
   - Visited `http://coffely.thm` to generate logs.  
   - Accessed `http://coffely.thm/secret-flag.html` to retrieve logs containing the flag:  
     ```
     {COffely_Is_Best_iN_TOwn}
     ```  

#### **Rationale:**  
Monitoring web server logs allows for tracking requests, responses, and detecting anomalies such as unauthorized access or errors.

---

## **Lessons Learned**  

1. **Splunk Deployment**: Installing Splunk Enterprise on Linux/Windows is straightforward, but understanding its architecture (indexers, forwarders, ports) is critical for scaling deployments.  
2. **Log Forwarding**: Configuring **Universal Forwarders** is an essential skill for collecting logs from endpoints efficiently.  
3. **Data Ingestion**: Knowing which log sources to monitor (e.g., `syslog`, `Event Logs`, `IIS logs`) ensures comprehensive visibility into system and web activities.  
4. **CLI Proficiency**: Using Splunk CLI commands like `start`, `stop`, `add monitor`, and `search` streamlines the configuration and troubleshooting process.  
5. **Validation**: Testing configurations using tools like `logger` and direct searches ensures that logs are ingested correctly.  

---
