# Opensource SIEM Homelab Setup

## Objective
To set up a cost-effective and customizable SIEM home lab using open-source tools for hands-on experience in cybersecurity, enabling real-time monitoring, log analysis, and threat detection in a controlled environment.

### Skills Learned

- Log Collection & Ingestion        : Configuring Logstash/Filebeat to collect logs.
- Data Parsing & Processing         : Structuring and filtering raw log data.
- Log Analysis & Threat Detection   : Identifying suspicious activities through log correlation.
- Data Visualization & Dashboarding : Creating dashboards in Kibana for real-time monitoring.
- Incident Investigation            : Analyzing logs to detect and respond to security threats.

### Tools Used

- **Virtualization Software** To create and manage virtual machines.
    Examples: VirtualBox or VMware
- **Ubuntu (Linux OS)** The base operating system for hosting the SIEM tools.
- **Java** Required for running Elasticsearch and Logstash.
- **Elastic Stack (ELK)** The core SIEM solution for log management and analysis:
- **Elasticsearch** Stores and indexes logs.
- **Logstash**  Processes and filters logs.
- **Kibana** Visualizes logs and security events.
- **Filebeat** Collects and ships logs from various sources.
- **Log Generators** To create logs for testing and analysis.
- **Wireshark** for network traffic analysis.
    Examples: Sysmon (Windows), Linux Auditd, or manual log entry.

## Homelab Network Architecture
![Architecture](https://github.com/Abhijithprashanth/Abhijithprashanth/blob/main/Homelab%20Network%20Architecture)

## Steps

Below are the key steps taken in the setup of SIEM Homelab

### 1. Installation and Set up of Virtualization Software
The home lab will run on Virtual Machines (VM) to create a isolated environments. here i choose VirualBox we can also use VMware too.
 - By using below link you can download Virtual box
   
   https://www.virtualbox.org/wiki/Downloads

### 2. Set up Ubuntu Virtual Machine
Download the Ubuntu Server ISO from the official
 - Use below link

   https://ubuntu.com/download/server
   
Once installed use below queries to update packages
- sudo apt update &&
- sudo apt upgrade -y
![Upgrade and Update](https://github.com/Abhijithprashanth/Abhijithprashanth/blob/main/update%20upgrade.png)

### 3. Install Java
Elasticsearch, which is part of the ELK stack, requires Java. Install the OpenJDK package
- sudo apt install openjdk-11-jdk -y
To verify installation use,
- java -version

![Java Version](https://github.com/Abhijithprashanth/Abhijithprashanth/blob/main/java%20ver.png)

### 4. Install Elasticsearch
Elasticsearch is the search engine and data store for your SIEM lab.
- Step 1 : Download and install the public signing key
  wget -qO – https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add –
- Step 2 : Install the APT repository
  sudo sh -c ‘echo “deb https://artifacts.elastic.co/packages/7.x/apt stable main” > /etc/apt/sources.list.d/elastic-7.x.list’
- Step 3 : Update your package list
  sudo apt update
- Step 4 : Install Elasticsearch
  sudo apt install elasticsearch -y
  
Configure Elasticsearch
- Step 1 : Open the configuration file
  sudo nano /etc/elasticsearch/elasticsearch.yml
- Step 2 : Set the following parameters:
  network.host: 127.0.0.1
- Step 3 : Save and close the file.
- Step 4 : Start and enable Elasticsearch to start on boot
  sudo systemctl start elasticsearch
  sudo systemctl enable elasticsearch


### 5. Install Logstash
Create a configuration file
sudo nano /etc/logstash/conf.d/01-logstash.conf

- Add following content 

input {

 tcp {

 port => 5000

 }

 udp {

 port => 5000

 }

 }

filter {

grok {

match => { “message” => “%{SYSLOGLINE}” }

}

date {

match => [ “timestamp”, “MMM d HH:mm:ss”, “MMM dd HH:mm:ss” ]

}

}

output {

elasticsearch {

hosts => [“localhost:9200”]

}

stdout { codec => rubydebug }

}

- Start and enable Logstash
  sudo systemctl start logstash
  sudo systemctl enable logstash

### 6. Install Kibana
Kibana is the visualization tool that allows you to explore the logs stored in Elasticsearch.

- Step 1 : Install Kibana
  sudo apt install kibana -y
- Step 2 : Configure Kibana
           - Open the Kibana configuration file
             sudo nano /etc/kibana/kibana.yml
           - Set the following configuration to bind Kibana to localhost
             server.host: “localhost”
           - Save and close the file.
           - Start and enable Kibana
             sudo systemctl start kibana
             sudo systemctl enable kibana

### 7. Install Filebeat
Filebeat is an agent installed on endpoints to collect and forward log data to Logstash or Elasticsearch.

- Step 1 : Install Filebeat
  sudo apt install filebeat -y
- Step 2 : Configure Filebeat
           - Open the configuration file
             sudo nano /etc/filebeat/filebeat.yml
           - Add the following to send logs to Logstash
             output.logstash:
             hosts: [“localhost:5000”]
           - Start and enable Filebeat
             sudo systemctl start filebeat
             sudo systemctl enable filebeat

### 8. Generate Logs for Analysis
To simulate log generation in your SIEM Install syslog on the secondary VM
sudo apt install rsyslog

Configure rsyslog to send logs to Logstash on the primary VM by adding the following lines to /etc/rsyslog.conf:

*.* @<your_elasticsearch_vm_ip>:5000

### 9. Testing SIEM Setup
To test the setup, log into Kibana and check if the logs are being received and indexed correctly by Elasticsearch
- Navigate to the Discover tab in Kibana.
- Select your index patterns (which should be the logs from Logstash).
- Explore the logs, filter based on specific patterns, and create visualizations.

### 10. Add Security Rules and Alerts
To extend your SIEM lab’s capabilities, integrate Wazuh, an open-source security monitoring platform that includes alerting and threat detection capabilities.
sudo apt install wazuh-manager wazuh-api

Configure Wazuh to send alerts to Logstash, and use Kibana to visualize these alerts.
