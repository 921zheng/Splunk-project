# Splunk-project

## Data Source: 
Udemy course: cisco_ironport_web.log/MOCK_DATA.csv/WWW1(access.log & secure.log)/WWW2(access.log & secure.log)/WWW3(access.log & secure.log)

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
Using the buttercupgames dataset, produce the following KOs for me as a stakeholder:

5 Dashboards that show me:
Buying trends
Site Visitor trends
Website Traffic over time
Potential website improvements
Security vulnerabilities/concerns within the website

At least 3 reports, each with a schedule
At least one related to security
At least one related to business insight
One of your choice

Create at least 3 alerts
Monitoring areas of your choice

Bonus points if you:
Use macros
Use field aliases
Implement Drilldowns

## Process:
### Ingest data:
upload cisco_ironport_web.log and 6 files from www1,www2,www3 -access.log and secure. log.

### Set Up Field Extractions
cisco:wsa
(Cisco IronPort Web Security Appliance)

timestamp
client_ip
http_method
url
response_code
bytes_transferred
user_agent
referrer
access_combined (Web Access Logs)

timestamp
client_ip
http_method
url
status
bytes
referrer
user_agent
linux_secure (Security Logs)

timestamp
user
action
src_ip
dest_ip
result

### Create the Dashboards
Dashboard 1: Buying Trends
Focus: This dashboard will track user activities that indicate buying behavior.

Sources: Primarily from the cisco_ironport_web.log (sourcetype cisco:wsa:squid) and access logs (access_combined).

Example Search Queries:

Identify URLs associated with purchases:

index=web sourcetype=access_combined url="*purchase*" OR url="*checkout*"
| stats count by url, client_ip 
| sort -count

Top Products Viewed:

index=web sourcetype=access_combined url="*product*" 
| stats count by url 
| sort -count

Visualizations:

Bar chart: Top products purchased or viewed.
Time chart: Purchase activity over time.

Dashboard 2: Site Visitor Trends
Focus: Track the number of unique visitors, sessions, and general site interaction.

Sources: Primarily from the cisco_ironport_web.log and access logs (access_combined).

Example Search Queries:

Unique Visitors Over Time:
spl
Copy code
index=web sourcetype=access_combined 
| timechart span=1h dc(client_ip) as Unique_Visitors
Session Duration:
spl
Copy code
index=web sourcetype=access_combined 
| transaction client_ip maxpause=30m 
| stats avg(duration) as Avg_Session_Duration
Visualizations:

Line chart: Unique visitors over time.
Table: Average session duration.
Dashboard 3: Website Traffic Over Time
Focus: Analyze how traffic to the website varies over time.

Sources: From all access logs (access_combined), this may include all web servers (web1, web2, web3).

Example Search Queries:

Traffic Volume Over Time:
spl
Copy code
index=web sourcetype=access_combined 
| timechart span=1h count as Traffic_Count
Traffic by Host:
spl
Copy code
index=web sourcetype=access_combined 
| timechart span=1h count by host
Visualizations:

Time chart: Overall traffic over time.
Stacked area chart: Traffic per host over time.
Dashboard 4: Potential Website Improvements
Focus: Identify slow-loading pages, errors, and other areas where user experience might be improved.

Sources: Primarily from the access_combined logs.

Example Search Queries:

Slow Pages:
spl
Copy code
index=web sourcetype=access_combined 
| stats avg(response_time) as Avg_Response_Time by url 
| sort -Avg_Response_Time
HTTP Errors (4xx/5xx status codes):
spl
Copy code
index=web sourcetype=access_combined status>=400 
| stats count by status, url 
| sort -count
Visualizations:

Bar chart: Pages with the highest average response time.
Pie chart: Distribution of HTTP status codes.
Dashboard 5: Security Vulnerabilities/Concerns
Focus: Monitor and report on security-related incidents, such as failed login attempts, unauthorized access, etc.

Sources: From linux_secure and cisco:wsa:squid.

Example Search Queries:

Failed Login Attempts:
spl
Copy code
index=security sourcetype=linux_secure action="failed login" 
| stats count by user, src_ip 
| sort -count
403/404 Errors:
spl
Copy code
index=web sourcetype=cisco:wsa:squid status_code=403 OR status_code=404 
| stats count by client_ip, url 
| sort -count
Unauthorized Access:
spl
Copy code
index=security sourcetype=linux_secure action="unauthorized access"
| stats count by user, src_ip, dest_ip
| sort -count
Visualizations:

Table: Users with the most failed login attempts.
Map: Geographic distribution of unauthorized access attempts.
Bar chart: Most common URLs resulting in 403/404 errors.
3. Building the Dashboards
Create a New Dashboard:

Go to the Search & Reporting app in Splunk.
Click on Dashboards and then Create New Dashboard.
Name the dashboard based on its focus (e.g., "Buying Trends").
Add Panels:

Click on Add Panel and choose New from Search.
Input the search query for each panel and configure the visualization type (e.g., bar chart, line chart).
Customize panel titles and descriptions.
Customize and Save:

Arrange the panels as needed on the dashboard.
Save the dashboard and adjust any additional settings like permissions.
4. Iterate and Refine
Review Data: Regularly check the dashboards to ensure they provide accurate and actionable insights.
Set Alerts: If certain metrics require immediate attention, set up alerts based on the dashboard data (e.g., spikes in failed login attempts).

### Create reports

1. Security Report: Failed Login Attempts
Purpose: Monitor failed login attempts to identify potential security threats, such as brute-force attacks.


index=security sourcetype=linux_secure action=failure
| stats count by user, src_ip
| sort -count
Steps to Create and Schedule:
Run the Search: Use the above query to identify failed login attempts.
Save as Report:
After running the query, click on "Save As" > "Report".
Name it something like "Failed Login Attempts by User and Source IP".
Schedule the Report:
In the report creation screen, select "Schedule".
Set the frequency (e.g., daily, hourly).
Choose the time of day to run the report.
Set up an email alert or notification if the report identifies a high number of failures.
Visualization: Optionally, add a bar chart or table to visualize the counts.
2. Business Insight Report: Top Products or Services Accessed
Purpose: Identify which products or services are most frequently accessed, giving insight into user behavior and business trends.

Search Query:
spl
Copy code
index=web sourcetype=access_combined
| rex field=url "/(?<product>[^/]+)\?"
| stats count by product
| sort -count
Steps to Create and Schedule:
Run the Search: Use the above query to identify the most accessed products or services.
Save as Report:
Name it something like "Top Accessed Products".
Schedule the Report:
Schedule it to run weekly or monthly to track trends over time.
Consider setting up alerts if access to specific products increases or decreases significantly.
Visualization: A bar chart showing the most accessed products/services by count.
3. Custom Report: Website Traffic Overview
Purpose: Monitor overall website traffic trends, helping you understand peak usage times and potential issues.

Search Query:
spl
Copy code
index=web sourcetype=access_combined
| timechart span=1h count by host
Steps to Create and Schedule:
Run the Search: Use the query to create an overview of traffic over time, broken down by host (e.g., www1, www2, www3).
Save as Report:
Name it something like "Hourly Website Traffic Overview".
Schedule the Report:
Schedule it to run daily or every few hours.
Useful to understand traffic patterns and identify potential load or availability issues.
Visualization: Use a line chart or area chart to visualize traffic over time.
How to Schedule Reports in Splunk:
After Running Your Query:

Click on "Save As" > "Report".
Fill out the report name, description, and any other details.
Click on "Schedule".
Set Schedule Parameters:

Frequency: Choose when and how often the report should run (e.g., daily at 6 AM).
Time Range: Define the time range the report should cover (e.g., "Last 24 hours").
Alert Actions: Choose if you want to send an email or trigger another alert when the report runs.
Review and Save: Finalize and save the scheduled report.

Summary of Reports:
Security Report: Failed Login Attempts (Scheduled Daily).
Business Insight Report: Top Accessed Products (Scheduled Weekly).
Custom Report: Website Traffic Overview (Scheduled Daily or Hourly).

### Create 3 Alerts

Alerts are essential for monitoring critical events and taking proactive measures. Below are three examples of alerts you can set up:

Alert 1: Excessive Failed Login Attempts (Security Monitoring)
Purpose: Detect potential brute-force attacks or unauthorized access attempts by monitoring excessive failed login attempts within a short period.

Search Query:
spl
Copy code
index=security sourcetype=linux_secure action=failure
| stats count by src_ip
| where count > 10

Alert Configuration:
Trigger Condition: When count > 10 (indicating more than 10 failed login attempts from the same IP within a specific time).
Time Window: 10 minutes (adjust based on your environment).
Alert Actions: Send an email notification to the security team, trigger a webhook, or log the alert to a file.

Alert 2: High Website Response Time (Performance Monitoring)
Purpose: Identify when your website's response time exceeds acceptable thresholds, which may indicate performance issues.

Search Query:
spl
Copy code
index=web sourcetype=access_combined
| stats avg(response_time) as avg_response_time by host
| where avg_response_time > 2  // Adjust threshold as needed

Alert Configuration:
Trigger Condition: When avg_response_time > 2 seconds.
Alert Actions: Send an email, create a ticket in your incident management system, or send a Slack message.


Alert 3: Unusual Spike in Traffic (Traffic Monitoring)
Purpose: Detect unusual spikes in website traffic that could indicate a DDoS attack or viral content.

Search Query:
spl
Copy code

index=web sourcetype=access_combined
| timechart span=5m count
| eventstats avg(count) as avg_traffic
| where count > 2 * avg_traffic

Alert Configuration:
Trigger Condition: When count > 2 * avg_traffic (indicating traffic has doubled compared to the average).
Time Window: 15 minutes.
Alert Actions: Send an email or SMS, trigger a webhook to start mitigation steps.

2. Use Macros
Macros in Splunk are reusable search strings that help simplify your queries and ensure consistency.

Example Macro: Failed Login Attempts
Create a macro that returns failed login attempts from the security index.

Macro Definition:

Name: failed_login_attempts

index=security sourcetype=linux_secure action=failure

`failed_login_attempts` 


3. Use Field Aliases

Sourcetype, Source, or Host: Specify the sourcetype (like access_combined) that this alias applies to.
Original Field: Enter the original field name (e.g., client_ip).
Alias Field: Enter the new field name (e.g., src_ip).


index=your_index sourcetype=access_combined | stats count by src_ip
same as
index=your_index sourcetype=access_combined | stats count by client_ip


Steps to Implement Drilldowns in Splunk:
Create or Edit a Dashboard:

First, navigate to Dashboards from the Splunk homepage.
Choose an existing dashboard to edit or create a new one by clicking Create New Dashboard.
Add a Chart or Table Visualization:

Add a chart (e.g., bar, pie, or time chart) or table visualization that shows relevant data (like login failures by user).
For example:
spl
Copy code
index=security sourcetype=linux_secure action=failure | stats count by user | sort - count
Save the panel.
Enable Edit Mode for the Dashboard:

Click Edit at the top right of the dashboard to enable edit mode.
Select the Visualization Panel:

Hover over the panel you want to add a drilldown to and click on the Edit icon (a pencil icon).
Set Up the Drilldown Action:

In the panel editing window, scroll down to find the Drilldown section.
By default, Splunk might have "Enable drilldown" checked. Here you can specify the drilldown behavior:
Link to Search: This option will take you to a search page pre-filtered by the clicked value.
Link to Dashboard: This allows you to link to another dashboard.
Link to Custom URL: If you want to open an external page.
Set Tokens: For more advanced drilldowns, you can set tokens that will filter data within the same dashboard.
Example Drilldown - Link to Search:

Suppose you want to see detailed logs when clicking on a specific user.
Choose Link to Search.
Under Search String, you can specify a query like:
spl
Copy code
index=security sourcetype=linux_secure action=failure user=$click.value$
$click.value$ automatically picks the clicked value (in this case, the username) and passes it to the search.
Save Your Changes:

Once configured, click Apply or Save to keep your drilldown settings.
Test the Drilldown:

Now, view your dashboard in normal mode (exit edit mode).
Click on a data point (like a specific user in a chart) and you should be redirected to a search page or a new panel with detailed results filtered based on your click.

