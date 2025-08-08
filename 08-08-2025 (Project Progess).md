Project Status Report: UEBA Platform MVP
Date: August 3, 2025
Current Phase: Completion of Data Pipeline

1. Overall Project Goal
The objective is to build a functional User & Entity Behavior Analytics (UEBA) platform from scratch. The Minimum Viable Product (MVP) will ingest security logs from a Windows Active Directory (AD) server, analyze them to detect anomalies, and display actionable insights on a web-based dashboard.

2. Completed Milestones & Key Activities
We have successfully completed the first two major phases of the project: setting up the core infrastructure and building the data pipeline.

Milestone 1: ELK Server Setup (The Central Brain)
The first major task was to build and configure a dedicated server to host the ELK Stack (Elasticsearch, Logstash, Kibana).

Server Environment:

Operating System: Ubuntu Server 22.04

Key Commands & Procedures:

Java Installation: Installed OpenJDK 11, a prerequisite for Elasticsearch and Logstash.

sudo apt update && sudo apt install -y openjdk-11-jdk

Elastic Repository Setup: Added the official Elastic software repository to the server.

wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list

Elasticsearch Installation & Configuration:

Installed the package: sudo apt install -y elasticsearch

Edited the configuration file: sudo nano /etc/elasticsearch/elasticsearch.yml

Errors & Resolutions:

Problem: Service failed to start (status=70).

Solution 1 (Configuration): Modified elasticsearch.yml to run as a single node and disabled the default security for simplicity (discovery.type: single-node, xpack.security.enabled: false).

Solution 2 (Permissions): Corrected directory ownership so the service could write to its data folders (sudo chown -R elasticsearch:elasticsearch /var/lib/elasticsearch /var/log/elasticsearch).

Solution 3 (Memory): Adjusted the Java memory allocation, a common fix for VMs, by creating /etc/elasticsearch/jvm.options.d/jvm-heap-size.options with -Xms1g and -Xmx1g.

Kibana Installation & Configuration:

Installed the package: sudo apt install -y kibana

Edited the configuration file: sudo nano /etc/kibana/kibana.yml

Error & Resolution:

Problem: Could not access the Kibana webpage ("Connection Refused").

Solution: Changed server.host from the default "localhost" to "0.0.0.0" to allow connections from other machines on the network.

Opened the firewall port: sudo ufw allow 5601/tcp

Logstash Installation:

Installed the package: sudo apt install -y logstash

Milestone 2: Data Pipeline Configuration (Connecting the Dots)
The second major task was to connect your Active Directory server to the ELK Stack so that logs could flow between them.

Key Commands & Procedures:

Winlogbeat Installation (on Windows AD Server):

Downloaded and extracted Winlogbeat to C:\Program Files\Winlogbeat.

Edited the configuration file winlogbeat.yml.

Winlogbeat Configuration & Troubleshooting:

Problem 1: Could not run the installation script (.\install-service-winlogbeat.ps1) due to security policy.

Solution 1: Bypassed the execution policy for the current session: Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass.

Problem 2: The Winlogbeat service failed to start.

Solution 2: Debugging revealed a panic: name Security already used error. This was caused by having duplicate entries for the "Security" log in winlogbeat.yml. The file was corrected to only list the Security log once.

Final Configuration: Pointed the output to Logstash on the ELK server: output.logstash: hosts: ["<your_elk_server_ip>:5044"].

Logstash Configuration (on ELK Server):

Created a new configuration file: sudo nano /etc/logstash/conf.d/02-winlogbeat-input.conf.

Configured Logstash to listen for Beats on port 5044 and output the received data to Elasticsearch in an index named winlogbeat-*.

Restarted the service to apply changes: sudo systemctl restart logstash.service.

3. Current Status: Where We Are Now
We have successfully completed the entire data engineering phase of the project.

You have a fully operational ELK Stack running on a dedicated Ubuntu server.

You have a live, real-time data pipeline shipping security logs from your Active Directory server to your ELK server.

You have verified that these logs are visible and correctly indexed in Kibana.

You understand the scope of the data you are collecting (central authentication events) versus what you are not (local laptop events).

This is the foundational work upon which all the "smart" features of the UEBA platform will be built.

4. What is Remaining: The Path to MVP
The next phases of the project involve building the analytics and user interface on top of the data you are now collecting.

Phase 3: Analytics Engine Development (The "Brain")

Task: Write Python scripts that will run on the ELK server.

Goal: These scripts will connect to Elasticsearch, read the log data, and perform the core UEBA logic:

Behavioral Baselining: For each user, calculate their "normal" behavior (e.g., typical login times, common workstations).

Anomaly Detection: Compare new logs against the baseline to find suspicious deviations.

Risk Scoring: Assign and update a risk score for each user based on their anomalous activity.

Phase 4: Frontend Dashboard Development (The "Face")

Task: Build a simple web application (likely using React).

Goal: Create a user interface for the security analyst that:

Displays a dashboard of the highest-risk users.

Allows the analyst to click on a user to see a detailed timeline of their alerts.

Phase 5: Integration, Testing, and MVP Delivery

Task: Connect the frontend to the backend analytics, perform end-to-end testing, and prepare a final demonstration of the working product.