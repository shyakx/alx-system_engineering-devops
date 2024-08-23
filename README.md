#Postmortem: Outage of Web Application Authentication Service
Issue Summary
Duration: August 22, 2024, from 03:15 UTC to 06:45 UTC (3 hours 30 minutes)

Impact: The outage affected the authentication service of our web application, preventing approximately 85% of users from logging in or signing up. Users experienced "500 Internal Server Error" messages when attempting to authenticate. Existing sessions remained active, so only new logins were affected.

Root Cause: The issue was caused by a misconfiguration in a recent deployment of the authentication service, which resulted in an incorrect environment variable that pointed the service to a non-existent database endpoint.

Timeline
03:15 UTC - Issue detected through automated monitoring alerts indicating a spike in "500 Internal Server Error" responses on the authentication service.
03:20 UTC - The on-call engineer received the alert and began investigating the issue by checking server logs and application metrics.
03:30 UTC - Initial suspicion was a database connectivity issue; the database team was notified, and connectivity checks began.
04:00 UTC - Database connectivity was confirmed to be stable. Focus shifted to the recent deployment as a potential cause.
04:20 UTC - Deployment logs were reviewed, and a misconfigured environment variable was identified.
04:30 UTC - The incident was escalated to the DevOps team for a hotfix.
05:15 UTC - A corrected configuration was deployed, but issues persisted, indicating additional factors.
05:45 UTC - Further investigation revealed a cached configuration on several instances preventing the new environment variable from taking effect.
06:00 UTC - All instances were restarted to clear the cache.
06:45 UTC - Full service was restored, and the authentication service was functioning normally.
Root Cause and Resolution
The root cause of the outage was a misconfigured environment variable introduced during a recent deployment. Specifically, the variable responsible for pointing the authentication service to its database was incorrectly set to a staging endpoint that did not exist in the production environment. This caused the authentication requests to fail, resulting in "500 Internal Server Error" messages for users trying to log in or sign up.

The issue was fixed by correcting the environment variable in the deployment configuration. Initially, this did not resolve the issue due to a cached configuration on several service instances. Once the cache was cleared by restarting these instances, the service resumed normal operation.

Corrective and Preventative Measures
Improvements/Fixes:

Deployment Process Review: The deployment process will be reviewed and updated to include a more thorough verification of environment variable configurations before pushing changes to production.
Environment Variable Management: Implement a centralized configuration management tool to avoid manual misconfigurations.
Cache Invalidation: Establish better practices for cache invalidation to ensure that configuration changes take effect immediately across all instances.

#TODO List:

Patch Deployment Pipeline: Update the deployment pipeline scripts to include environment variable validation checks.
Centralized Configurations: Set up and migrate all environment variables to a centralized configuration management system like Consul or etcd.
Enhanced Monitoring: Add monitoring for environment variable mismatches and unexpected service endpoint changes.
Documentation Update: Update the runbooks to include steps for checking and invalidating caches during incident resolution.
Training: Conduct a training session for all engineers on best practices for configuration management and deployment processes.
By implementing these corrective measures, we aim to reduce the likelihood of similar incidents occurring in the future and improve the overall resilience of our system.
