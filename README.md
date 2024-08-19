# Splunk-project

## Data Source: 
Udemy course: 
cisco_ironport_web.log
MOCK_DATA.csv
WWW1(access.log & secure.log)
WWW2(access.log & secure.log)
WWW3(access.log & secure.log)

## Install Splunk:

Use docker to open splunk

start container:
docker start splunk-neww

stop container: 
docker stop splunk-neww

link: http://localhost:8000/
user:admin 
ps:yearmonthday


## Objectives:
Using the buttercupgames dataset, produce the following KOs for a stakeholder:

5 Dashboards:
* Buying trends:
Site Visitor trends
Website Traffic over time
Potential website improvements
Security vulnerabilities/concerns within the website

* At least 3 reports, each with a schedule:
At least one related to security
At least one related to business insight
One of your choice

* Create at least 3 alerts:
Monitoring areas of your choice

* Bonus points:
Use macros
Use field aliases
Implement Drilldowns

## Process:
### Ingest data:
upload cisco_ironport_web.log and 6 files from www1,www2,www3 -access.log and secure.log.

### Set Up Field Extractions
reasons: log is unstructured data (and fields are not autimatically identified), field extractions can be used effectively for searches, analysis, and visualizations.

* cisco:wsa
(Cisco IronPort Web Security Appliance)
![Screen Shot 2024-08-16 at 11 04 36](https://github.com/user-attachments/assets/5c026e84-d116-46e9-81c5-96c5d71b4604)

timestamp
client_ip
http_method
url
status
bytes
user_agent
referrer

* access_combined (Web Access Logs)
![Screen Shot 2024-08-16 at 11 02 10](https://github.com/user-attachments/assets/e3b814de-ff89-48d2-b9ce-25e13fb7062f)

timestamp
client_ip
http_method
url
status
bytes
referrer
user_agent

*linux_secure (Security Logs)
![Screen Shot 2024-08-16 at 11 05 12](https://github.com/user-attachments/assets/7281425c-292d-48e4-9fe3-6bce661d71b7)

timestamp
user
action
src_ip
dest_ip
result

### Create the Dashboards

#### Dashboard 1: Buying Trends

Focus: This dashboard will track user interactions by counting occurrences of each URL .

Example Search Queries:


index=web sourcetype=access_combined OR sourcetype=cisco:wsa:squid
| stats count by url
| sort -count

![Screen Shot 2024-08-16 at 12 02 08](https://github.com/user-attachments/assets/1a778492-0157-453e-b164-391cd2ebbcb6)


#### Dashboard 2: Site Visitor Trends

Focus: Track the number of unique visitors, sessions, and general site interaction.

Example Search Queries:

Unique Visitors Over Time:

index=web sourcetype=access_combined 
| timechart span=1h dc(client_ip) as Unique_Visitors

![Screen Shot 2024-08-16 at 12 03 45](https://github.com/user-attachments/assets/9ec031e6-0220-4ffb-b173-d766b86da8b8)


index=web sourcetype=access_combined 
| transaction client_ip maxpause=30m 
| stats avg(duration) as Avg_Session_Duration

![Screen Shot 2024-08-16 at 12 07 32](https://github.com/user-attachments/assets/f5039df1-50a8-4cd3-8bfe-1a2f1d02516b)

#### Dashboard 3: Website Traffic Over Time
Focus: Analyze how traffic to the website varies over time.

Example Search Queries:

Traffic Volume Over Time:

index=web sourcetype=access_combined 
| timechart span=1h count as Traffic_Count

![Screen Shot 2024-08-16 at 12 16 25](https://github.com/user-attachments/assets/e955a069-41b8-404e-b2bf-98bff538dc05)


#### Dashboard 4: Potential Website Improvements
Focus: Identify slow-loading pages, errors, and other areas where user experience might be improved.

Example Search Queries:

HTTP Errors (4xx/5xx status codes):

index=web sourcetype=access_combined status>=400 
| stats count by status
| sort -count

![Screen Shot 2024-08-16 at 12 28 36](https://github.com/user-attachments/assets/4f4f96da-a921-4cd3-bacf-2d57ebcd827f)


#### Dashboard 5: Security Vulnerabilities/Concerns
Focus: Monitor and report on security-related incidents, such as failed login attempts, unauthorized access, etc.

Example Search Queries:

Failed Login Attempts:

index=security sourcetype=linux_secure action=failure | stats count by user
| sort -count

![Screen Shot 2024-08-16 at 12 32 40](https://github.com/user-attachments/assets/3d9a270f-2899-44f0-ad32-944766f8c4c2)

### Create reports

#### 1. Security Report: Failed Login Attempts

Purpose: Monitor failed login attempts to identify potential security threats, such as brute-force attacks.

index=security sourcetype=linux_secure action=failure
| stats count by user, src_ip
| sort -count

Schedule the Report:
In the report creation screen, select "Schedule".
Set the frequency (daily 10am).
Set up an email notification if the report identifies a high number of failures.

#### 2. Business Insight Report: Top Products or Services Accessed

Purpose: Identify which products or services are most frequently accessed, giving insight into user behavior and business trends.

index=web sourcetype=access_combined
| rex field=url "/(?<product>[^/]+)\?"
| stats count by product
| sort -count

Schedule the Report:
Schedule it to run weekly or monthly to track trends over time.
Consider setting up alerts if access to specific products increases or decreases significantly.

#### 3. Custom Report: Website Traffic Overview
Purpose: Monitor overall website traffic trends, helping you understand peak usage times and potential issues.

index=web sourcetype=access_combined
| timechart span=1h count by host

Schedule the Report:
Schedule it to run daily or every few hours.
Useful to understand traffic patterns and identify potential load or availability issues.

Frequency: Choose when and how often the report should run (e.g., daily at 6 AM).
Time Range: Define the time range the report should cover (e.g., "Last 24 hours").
Alert Actions: Choose if you want to send an email or trigger another alert when the report runs.
Review and Save: Finalize and save the scheduled report.

### Create 3 Alerts

#### Alert 1: Excessive Failed Login Attempts (Security Monitoring)
Purpose: Detect potential brute-force attacks or unauthorized access attempts by monitoring excessive failed login attempts within a short period.

index=security sourcetype=linux_secure action=failure
| stats count by src_ip
| where count > 10

Alert Configuration:
Trigger Condition: When count > 10 (indicating more than 10 failed login attempts from the same IP within a specific time).
Time Window: 10 minutes (adjust based on your environment).
Alert Actions: Send an email.

#### Alert 2: High Website Response Time (Performance Monitoring)
Purpose: Identify when your website's response time exceeds acceptable thresholds, which may indicate performance issues.

index=web sourcetype=access_combined
| stats avg(response_time) as avg_response_time by host
| where avg_response_time > 2  // Adjust threshold as needed

Alert Configuration:
Trigger Condition: When avg_response_time > 2 seconds.
Alert Actions: Send an email.

#### Alert 3: Unusual Spike in Traffic (Traffic Monitoring)
Purpose: Detect unusual spikes in website traffic that could indicate a DDoS attack or viral content.

index=web sourcetype=access_combined
| timechart span=5m count
| eventstats avg(count) as avg_traffic
| where count > 2 * avg_traffic

Alert Configuration:
Trigger Condition: When count > 2 * avg_traffic (indicating traffic has doubled compared to the average).
Time Window: 15 minutes.
Alert Actions: Send an email.

### Bonus 

#### 1. Use Macros

Process: setting-advanced search

Name: failed_login_attempts

index=security sourcetype=linux_secure action=failure


#### 2. Use Field Aliases

Sourcetype, Source, or Host: Specify the sourcetype (like access_combined) that this alias applies to.
Original Field: Enter the original field name (e.g., client_ip).
Alias Field: Enter the new field name (e.g., src_ip).


index=your_index sourcetype=access_combined | stats count by src_ip
same as
index=your_index sourcetype=access_combined | stats count by client_ip


#### 3. Steps to Implement Drilldowns in Splunk:

Process: 

First, navigate to Dashboards from the Splunk homepage.
Choose an existing dashboard to edit.

Click Edit at the top right of the dashboard to enable edit mode.
Select the Visualization Panel:

Hover over the panel you want to add a drilldown to and click on the Edit icon (a pencil icon).
Set Up the Drilldown Action:

In the panel editing window, scroll down to find the Drilldown section.
By default, Splunk might have "Enable drilldown" checked. Here you can specify the drilldown behavior:
Link to Search: This option will take you to a search page pre-filtered by the clicked value.
Link to Dashboard: This allows you to link to another dashboard.
Link to Custom URL: If you want to open an external page.


