# 0x19-postmortem
Postmortem: Website Outage Due to Database Connection Issue
Issue Summary
Duration: The outage occurred on August 16th, 2024, from 14:10 PM to 16:40 PM UTC (2 hours 30 minutes).
Impact: Our e-commerce website experienced intermittent connection issues, causing significant slowdowns in the checkout process. 75% of users were unable to complete transactions, and others experienced delayed responses when navigating the site. Approximately 60% of revenue was lost during the outage period.
Root Cause: A misconfigured database connection pool led to resource exhaustion, which blocked new connections and triggered cascading failures throughout the system.

Timeline
14:10 PM UTC - Monitoring alerts from New Relic indicate increased latency in the checkout process.
14:20 PM UTC - On-call engineer begins investigating latency issues in the web application.
14:25 PM UTC - Assumption made that the issue is related to increased traffic. Traffic management tools and CDN logs are reviewed.
14:40 PM UTC - Alerts continue, with latency worsening. An issue with third-party payment providers is suspected and escalated to the Payments team.
15:00 PM UTC - After further investigation, payment processing logs appear normal. Focus shifts back to application server logs and database monitoring.
15:20 PM UTC - Database connection pool saturation is discovered. Connection limits are maxed out, and new requests are failing.
15:40 PM UTC - Database administrator identifies the issue as a configuration error in the connection pool settings.
16:10 PM UTC - The database connection pool is reconfigured, and affected services are restarted.
16:40 PM UTC - All services are back to normal, and performance is restored.
Root Cause and Resolution
Root Cause:
The issue was traced back to a misconfiguration of the database connection pool, which had an insufficient maximum connection limit. As traffic increased throughout the day, the pool's connection limit was reached quickly. Once the pool was saturated, additional connection requests were queued, causing high latency. Eventually, all available connections were in use, resulting in connection failures across the application. This caused a bottleneck that propagated throughout the service, particularly during the checkout process.

Resolution:
Once the root cause was identified, the database administrator increased the maximum number of allowed connections in the pool. Additionally, idle connection timeouts were adjusted to release unused connections faster. The web application services were then restarted to clear the backlog of pending requests. The change was deployed across the production environment without further disruption.

Corrective and Preventative Measures
Improvements and Fixes:

Improve Database Connection Pool Management: Regularly review connection pool configurations to ensure they align with the current load.
Monitoring Enhancements: Implement specific monitoring for database connection saturation, ensuring alerts are triggered before reaching critical thresholds.
Traffic Scaling: Improve load testing procedures to ensure that connection pool limits can handle sudden traffic spikes.
Graceful Degradation: Introduce features that allow for more graceful degradation of service when database issues occur, such as providing cached data when the database is unreachable.
Action Items:

Increase database connection pool limits in production to match projected traffic growth.
Add detailed monitoring for connection pool utilization and set up automatic alerts for early detection.
Implement automated scaling solutions for both application and database servers to handle spikes in traffic.
Review and update load testing scripts to simulate higher traffic scenarios.
Schedule a knowledge-sharing session between engineering and database administration teams to better understand capacity planning.
By addressing these areas, we will be better prepared to prevent similar issues in the future and improve overall service reliability.
