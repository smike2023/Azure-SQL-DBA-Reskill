# SQL DBA Hands-On Lab Environment Guide

## Executive Summary and Lab Architecture Overview

This guide designs a reproducible, end-to-end lab environment that builds real-world database administration skills across three deployment scenarios: Azure-free-tier, local Windows virtual machine (VM), and local containerized SQL Server on Docker. The Azure path teaches platform-as-a-service (PaaS) administration and monitoring with native tooling; the Windows VM path instills deep, instance-level administration, service accounts, and clustering concepts; the Docker path develops operational fluency with ephemeral environments, data persistence, and modern DevOps workflows.

The lab progresses from beginner to advanced through a consistent set of exercises: installing SQL Server and connecting with SQL Server Management Studio (SSMS), setting up backups and restores, building a high-availability Always On Availability Group (AG), implementing database refresh automation, conducting performance tuning with Extended Events, integrating monitoring, and performing security audits. The design emphasizes hands-on realism, minimal cloud spend (particularly on the Azure free tier), and strong safety controls so learners can practice restore paths and failovers without endangering production.

What you will build:
- An Azure SQL Database free-tier serverless instance and a local SQL Server instance (Windows VM or Docker container) for basic administration labs and PaaS-specific exercises.
- An Always On Availability Group on Windows Server Failover Clustering (WSFC), including endpoints, replicas, listeners, and read-scale routing.
- Backup and restore simulations in both full and bulk-logged recovery models, including point-in-time recovery (PITR) and restore verification.
- Database refresh automation using copy-only backups and PowerShell-driven restore workflows, including Azure free-tier alternatives.
- Performance tuning labs that capture workload with Extended Events (XE), interpret results, and validate improvements via counters and query analytics.
- Integrated monitoring using SSMS, Transact-SQL (T-SQL), performance counters, and optional operational integration (Azure Monitor alerts for the free-tier database).
- Security audits with T-SQL scripts that inspect logins, roles, permissions, encryption, audit objects, and surface dangerous configurations.
- Troubleshooting playbooks covering installation misconfigurations, WSFC quorum, AG failover, and container connectivity.

Skill progression and outcomes:
- Beginner: Install SQL Server, configure SSMS connectivity, understand recovery models, issue basic backups/restores, and create Extended Events sessions. 
- Intermediate: Build and manage Always On AGs, execute planned failovers, automate database refreshes, and tune queries using XE captures and performance counters.
- Advanced: Validate RTO/RPO targets in DR simulations, configure read-only routing, automate XE data collection, establish monitoring alerts (Azure Monitor), and harden security with least privilege and audit coverage.

Information gaps to plan around:
- Minimum local hardware specs for Always On AG and performance labs are not formally specified here; design assumes modern laptops or equivalent VMs with at least 8–16 GB RAM and dual+ vCPUs. 
- Azure free-tier quotas, throughput, and regional specifics change over time; confirm current values in Azure documentation at deployment.
- Licensing constraints and edition behavior for containerized SQL Server in production require review of current licensing terms.
- Step-by-step WSFC cluster creation in Windows Server 2019/2022 is referenced but not fully reproduced; this guide outlines prerequisites and operational checks and directs learners to authoritative documentation.

This blueprint is evidence-based and safe by design. It anchors installation decisions in official requirements, leverages free-tier allowances to teach cloud operations, and develops the muscle memory needed for reliable administration through repeatable, tested exercises.[^1][^2][^3]



## Lab Scenario Selection and Deployment Choices

Selecting the right scenario depends on your learning objectives, hardware budget, connectivity, and the skills you aim to cultivate. Azure SQL Database free tier emphasizes managed operations, backup monitoring, and cloud-native alerting. A local Windows VM mirrors production behaviors for clustering, service accounts, and instance-level configuration. Docker-based SQL Server accelerates environment spin-up and supports rapid iteration with portability and data persistence techniques.

Azure SQL Database free tier highlights:
- A serverless General Purpose database, free for the lifetime of the subscription, with monthly allowances for compute, storage, and backups.
- Auto-pause behavior when free limits are reached, with an option to continue with pay-as-you-go charges.
- PaaS maintenance (patching, backups, monitoring) is handled by Azure; perfect for learning query tuning, security basics, and monitoring with portal tools.[^2]

Local Windows VM highlights:
- Full control over instances, features, and clustering required for Always On AG exercises, including WSFC, endpoints, and listener creation.
- Ideal for backups/restores, granular security controls, service accounts, and advanced monitoring using SSMS, XE, and performance counters.[^1]

Docker-based SQL Server highlights:
- Fast setup with mssql container images, suitable for repeatable labs and ephemeral instances.
- Requires explicit data persistence via bind mounts or volume mounts; excellent for learning DevOps-style workflows and operational discipline around container logs, networking, and persistence.[^3][^17][^18][^19]

To orient choices, the matrix below summarizes trade-offs.

Table 1. Deployment scenarios matrix: Azure free-tier vs Windows VM vs Docker

| Dimension | Azure SQL Database free tier (PaaS, serverless) | Windows VM (on-prem/laptop) | Docker on Linux/Windows |
|---|---|---|---|
| Core learning goals | PaaS administration, monitoring, backups, security basics | Full instance control, clustering (WSFC/AG), service accounts, advanced diagnostics | Rapid environment spin-up, persistence, DevOps workflows, container troubleshooting |
| Pros | Fully managed; zero-cost allowances; Azure Monitor alerts; easy onboarding | Mirrors production HA/DR patterns; granular control; no external dependencies | Fast provisioning; portable; consistent; simple reset/restore for labs |
| Cons | Limited feature exposure (no direct WSFC/AG); network and region restrictions | Heavier setup; machine resource demands; OS patching | Data loss if not persisted; networking quirks; licensing constraints for production |
| Costs | Free within monthly allowances; pay-as-you-go optional | Hardware/power only | Host OS + container runtime |
| Feature support (AG, backups, XE, monitoring) | AGs not applicable; backups/restore via PaaS; XE-like via portal metrics; Azure Monitor alerts | Full AG; backups/restores; XE; SSMS; Performance Monitor | Backups via T-SQL; XE; SSMS from host; limited cluster features |
| Complexity | Low | Medium to High | Low to Medium |
| Reset/repeatability | High (delete/recreate) | Medium (snapshots/VM rollback) | High (stop/rm, re-run) |

The right answer is often “and,” not “or.” Running both a cloud and a local path cross-trains the mind for the realities of hybrid operations.



## Azure Free Tier Setup (Step-by-Step)

The Azure SQL Database free tier provides a serverless General Purpose database that is free for the lifetime of your subscription, with monthly allowances for compute (vCore seconds), storage, and backups. Once you reach the monthly compute limit, the database auto-pauses until the next month unless you opt into continuing with additional charges. Regional constraints apply at the subscription level for free databases.[^2]

Table 2. Azure SQL free-tier allowances and behaviors

| Item | Free-tier allowance/behavior |
|---|---|
| Compute | 100,000 vCore seconds per month |
| Data storage | Up to 32 GB per month |
| Backup storage | Up to 32 GB per month (LRS) |
| Max databases per subscription | 10 General Purpose free databases |
| Behavior on reaching limits | Auto-pause until next month, or continue with pay-as-you-go |
| Region | Fixed for all free databases in a subscription once selected |
| Free offer lifetime | Free for the lifetime of the subscription |

Prerequisites and plan:
- You need an Azure account and subscription. The free offer applies regardless of subscription type for the current offering; confirm any special eligibility constraints at sign-up.[^2]
- Select a region that will host all free-tier databases for this subscription. The region cannot be changed later for free databases in the same subscription.[^2]
- Plan to track usage to avoid unexpected charges if you opt to continue beyond free limits.

Create the free-tier database:
1) Open the Azure SQL hub and choose Create SQL Database. Apply the free offer banner. 
2) In Project details, create or select the resource group (for example, myFreeDBResourceGroup) and database name (for example, myFreeDB).
3) Create a new logical server (for example, myfreesqldbserver), choose Authentication (SQL and Microsoft Entra), and set a strong admin login/password. Select a region. 
4) In Compute + storage, keep the default Standard-series (Gen5), 2 vCores, up to 32 GB storage as applicable to free-tier usage.
5) Choose the behavior when limits are reached: Auto-pause (no charges) or Continue (pay-as-you-go).
6) In Networking, allow Azure services and resources and add your current client IP to enable access.
7) In Additional settings, load the AdventureWorksLT sample dataset (recommended for exercises) or leave blank.
8) Review + create and confirm the cost summary shows zero charges.[^2]

Querying and monitoring:
- Use the Query editor (preview) from the Azure portal to run ad hoc queries and validate operations.
- Monitor free-usage metrics on the database Overview and Metrics pages. Set Azure Monitor alert rules on the “Free amount remaining” metric to avoid surprises; for example, alert when less than 10,000 vCore seconds remain.[^2]
- To reset, delete the resource group. Confirm that free databases are not placed in elastic pools or failover groups, which are unsupported for the free offer.[^2]



## Local Alternatives: SQL Server on Windows VM

A Windows-based SQL Server installation is the most direct path to learning instance-level administration, Always On AGs, and advanced backup and monitoring techniques. Start by aligning your planned installation with SQL Server’s official hardware and software requirements.

System requirements and disk planning:
- Minimum memory: 1 GB for most editions (2 GB for Data Quality Server); Express editions may start at 512 MB, but increase memory for real workloads. Recommended memory starts at 4 GB and scales with database size.[^1]
- Disk space: At least 6 GB available during setup; feature-specific footprints vary. Plan for data, log, and tempdb separation.[^1]
- CPU: x64 1.4 GHz minimum; 2.0 GHz or faster recommended.[^1]
- OS support: Windows 10 1607+ and Windows Server 2016+ (including Server Core variants) are supported.[^1]

Table 3. SQL Server feature disk-space requirements (selected highlights)

| Feature group | Approximate disk space |
|---|---|
| Database Engine + Data files, Replication, Full-Text Search, DQS | ~1,480 MB |
| Database Engine with R Services (In-Database) | ~2,744 MB |
| Database Engine with PolyBase Query Service for External Data | ~4,194 MB |
| Analysis Services and data files | ~698 MB |
| Reporting Services | ~967 MB |
| All features (combined) | ~8,030 MB |
| Books Online content (downloaded) | ~200 MB |

Other planning considerations:
- Sector size: Native 512-byte and 4 KB sector sizes are supported. Larger sector sizes can cause errors; the ForcedPhysicalSectorSizeInBytes registry key may be required for advanced format disks.[^1]
- Install locations: Place tempdb and user data files on suitable storage; for cluster deployments, ensure path validity across all nodes.[^1]
- Domain controller: Installing on a domain controller is not recommended and has significant limitations; review the restrictions if your lab includes AD integration.[^1]

Installation methods:
- Installation Wizard (Setup) for interactive installation and feature selection.[^5]
- Command-line installs for unattended scenarios, useful when building repeatable lab images.[^5]
- SysPrep to generalize an instance for imaging and later configuration.[^5]
- Slipstream installs to incorporate updates at install time.[^5]

Post-install actions:
- Confirm instance connectivity via SSMS; verify TCP/IP is enabled if remote access is required.[^5]
- Configure Windows Firewall to allow SQL Server ports and SQL Browser as needed.[^5]
- Validate system configuration checks and review Setup logs if any warnings or errors occur.[^5]
- Install latest servicing updates and validate the installation.[^5]



## Local Alternative: SQL Server in Docker Containers

Docker provides a lightweight path to consistent SQL Server labs across operating systems, with minimal overhead for spin-up and teardown. The official SQL Server Linux images are available from the Microsoft Container Registry (MCR), with tags for major releases (2017, 2019, 2022, and 2025).[^19][^3]

Prerequisites:
- Docker Engine 1.8+ with the overlay2 storage driver (default on most platforms). 
- At least 2 GB RAM and 2 GB disk space available for the container image.[^3]
- Host CPU must be x86-64; emulation/translation environments are not supported.[^3]

Run your first container:
- Accept the End User License Agreement (EULA) by setting ACCEPT_EULA=Y.
- Set a strong SA password via MSSQL_SA_PASSWORD and map host port 1433 to container port 1433.
- Run the container in the background with a stable name and hostname.[^3][^18]

Example:
- docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=YourStrongPassword" -p 1433:1433 --name sql1 --hostname sql1 -d mcr.microsoft.com/mssql/server:2022-latest

Validate and connect:
- docker ps -a and inspect STATUS for “Up”.
- Inspect the SQL Server error log for “SQL Server is now ready for client connections”.
- Connect from the host using sqlcmd or SSMS, or use the VS Code MSSQL extension for agile development.[^3]

Operational notes and hardening:
- Persistence: Data in containers is deleted when stopped/removed unless persisted. Use volume or bind mounts to persist data and log files, and ensure OS-level permissions align with SQL Server’s requirements in the container.[^18]
- Change the SA password after startup; avoid leaving credentials visible in process environment variables. Consider disabling the SA login and using Windows or SQL logins with least privilege.[^3]
- For Linux hosts, configure the Docker daemon and storage driver per best practices; ensure kernel and filesystem support for container operations.[^22]

Table 4. Common Docker run options and environment variables

| Option/Variable | Purpose | Example |
|---|---|---|
| ACCEPT_EULA | Accept EULA (required) | -e "ACCEPT_EULA=Y" |
| MSSQL_SA_PASSWORD | Set SA password (strong, per policy) | -e "MSSQL_SA_PASSWORD=YourStrongPassword" |
| MSSQL_COLLATION | Optional collation override | -e "MSSQL_COLLATION=SQL_Latin1_General_CP1_CI_AS" |
| -p | Port mapping (host:container) | -p 1433:1433 |
| --name | Container name | --name sql1 |
| --hostname | Container hostname | --hostname sql1 |
| -d | Detached/daemon mode | -d |
| Volumes | Persist data/logs | -v sqlmdata:/var/opt/mssql |

Security best practices:
- Enforce strong SA password complexity and consider disabling the SA account post-provisioning.
- Prefer dedicated SQL logins or Windows authentication where feasible; restrict surface area to the minimum required for learning outcomes.[^3][^23]



## Always On Availability Groups: Hands-On Setup

Always On Availability Groups provide database-level protection and scale-out reads by synchronizing one or more databases across replicas. The core setup requires WSFC, a database mirroring endpoint on each instance, and careful attention to listener configuration for client connectivity.[^7]

Prerequisites and enabling:
- Each replica must run on a WSFC node. Review prerequisites and recommendations, including domain considerations, quorum planning, and file path alignment.[^7]
- Enable Always On Availability Groups in SQL Server Configuration Manager on each participating instance.[^7]

Endpoints:
- Confirm or create the database mirroring endpoint. Windows authentication is typical in domain environments; certificates can be used when domains are unavailable. Verify endpoint state and port accessibility.[^7]

Create the AG and add replicas:
- Use the New Availability Group Wizard, T-SQL, or PowerShell to create the AG and specify replicas and availability modes. For read-scale scenarios, asynchronous commit may be acceptable; for zero-data-loss guarantees across synchronized replicas, use synchronous commit with appropriate quorum.[^7]
- Strongly recommend creating a listener with a DNS name and virtual IP (VIP) to abstract connectivity from primary replica changes.[^7]

Join replicas and prepare secondaries:
- After creating the AG, join each secondary replica to the AG, restore backups of the target databases with NORECOVERY, and then join those databases to the AG to initiate data synchronization. The wizard can automate these steps if desired.[^7]

Listener and routing:
- Configure the listener’s DNS name and ports; supply connection strings using the listener for application connectivity.
- For read-scale, configure read-only routing to distribute read workloads to secondary replicas appropriately.[^7]

Failover modes and policies:
- Understand planned manual failover (no data loss when synchronous) and forced manual failover (possible data loss in DR scenarios). Set flexible failover policies to control automatic failover conditions.[^7]

Monitoring and management:
- Use the Always On dashboard in SSMS to visualize health, synchronization status, and failover readiness.
- Use T-SQL catalog views and DMVs to monitor AG state and performance counters for replicas and databases (e.g., log flush metrics).[^\[7]]

Table 5. AG configuration checklist

| Area | Configuration |
|---|---|
| WSFC and quorum | Cluster created, nodes healthy, quorum configured |
| Always On enabled | Enabled on each SQL instance |
| Endpoints | Endpoint exists, correct authentication, ports open |
| Replicas | Primary and secondary defined; availability mode set |
| Initial sync | Secondary databases restored with NORECOVERY; joined to AG |
| Listener | DNS name and VIP created; port defined; tested from clients |
| Backup preference | Automated backup preference set; backup jobs configured |
| Read routing | Read-only routing configured for read-scale |
| Failover modes | Replica failover modes aligned with HA/DR intent |
| Monitoring | Dashboard health green; counters and DMVs inspected |

Tip: In Azure VMs, Microsoft provides a manual configuration tutorial for AGs in a single subnet; adapt the network and quorum specifics accordingly if you build this lab in Azure.[^8]



## Database Refresh Automation Exercises

Automating database refreshes from production (or a representative source) to lower environments is a core operational skill. The goals are speed, repeatability, and safety. The most reliable pattern uses a copy-only full backup, transfers it securely, restores it with recovery (or with NORECOVERY if further logs will follow), and masks sensitive data to avoid compliance risks. Automate the pipeline with PowerShell and schedule it with SQL Server Agent.[^20][^21]

Conceptual flow:
- Source backup: Use copy-only backups to avoid breaking the production log chain.
- Transfer: Copy the backup file(s) to the target environment over a secure channel.
- Restore: Restore to the target database name with appropriate options (WITH REPLACE if needed), respecting recovery model and pending logs.
- Post-restore: Update statistics, run DBCC checks, and reapply any needed permissions or application-specific configurations.

PowerShell orchestration (indicative, adapt for your environment):
- Identify latest copy-only backup(s) from the source instance.
- Copy to the target server securely (e.g., SMB or cloud storage).
- Restore with the correct options; optionally restore subsequent log backups if required to reach the target timestamp.
- Kick off post-restore integrity checks and update statistics.[^21][^20]

Scheduling and safety:
- Wrap steps into a SQL Server Agent job or Windows scheduled task with retries, notifications on failure, and idempotent design.
- For AG environments, account for database participation; refresh targets may need to be removed from the AG or refreshed on a secondary if policy allows.

Azure free-tier alternative:
- If your target is Azure SQL Database (free tier), use Azure-native import/export mechanisms appropriate for serverless environments, or build an export/import workflow that respects free-tier allowances and auto-pause behavior. As the free offer is PaaS, traditional backup/restore patterns differ; treat it as a separate exercise using supported Azure tools and confirm limitations such as elastic pool and failover group incompatibility.[^2]

Table 6. Database refresh pipeline: steps, tools, and validation

| Step | Tooling | Validation |
|---|---|---|
| Identify source backup (copy-only) | T-SQL BACKUP COPY_ONLY | Backup history and checksums |
| Secure transfer | PowerShell (Copy-Item), secure channel | File integrity (hash) |
| Restore full | RESTORE DATABASE WITH REPLACE/NORECOVERY | Verify header and move files as needed |
| Apply logs (if needed) | RESTORE LOG with STOPAT | Point-in-time consistency |
| Post-restore | DBCC CHECKDB, UPDATE STATISTICS | Clean DBCC; statistics updated |
| Mask/scrub | Redaction scripts | PII rules applied |
| Automate/schedule | SQL Agent or Task Scheduler | Job history, alerts |

The essence of a good refresh pipeline is repeatability. Always test the entire chain in a non-production environment before running it for the first time in a sensitive context.[^20][^21]



## Backup/Restore Simulations

Backups are both a protection mechanism and a administrative tool. A robust strategy balances availability, data loss exposure, and cost. Restore planning is as important as backup planning, and both must be tested routinely.[^9]

Recovery models:
- Simple: Easy to manage, but limited point-in-time recovery (PITR) options because transaction logs are truncated. Suitable for small systems with less strict recovery requirements.
- Full: Enables PITR with transaction log backups; requires log backup discipline and storage planning. Preferred for production workloads.
- Bulk-logged: Minimizes log space usage for bulk operations; still requires log backups for recovery. Can limit PITR granularity; plan carefully.[^9]

Backup types and applications:
- Full backups capture the whole database. For very large databases, consider partial and file/filegroup backups.
- Differential backups capture changes since the last full; reduce restore time when combined with full.
- Log backups enable PITR under full or bulk-logged models.[^9]

Locations and verification:
- Back up to local disk for speed, to cloud storage for durability, or to tape for archival. Always verify backups using BACKUP CHECKSUM and RESTORE VERIFYONLY, and test restores.[^9]
- Record backup locations, device names, and expected restore times in an operations manual so that recovery is not a scavenger hunt.[^9]

Monitoring and overhead:
- Extended Events provide insight into long-running backup/restore operations; use short-lived sessions and guard disk space when enabling additional traces.[^9]

Table 7. Recovery models vs capabilities

| Capability | Simple | Full | Bulk-logged |
|---|---|---|---|
| PITR via log backups | No | Yes (granular) | Limited (depending on bulk operations) |
| Log backup frequency | N/A | Regular | Regular |
| Work-loss exposure | Higher (since last backup) | Minimal (with regular log backups) | Moderate (bulk ops can add risk) |
| Storage overhead | Lower | Higher (logs retained) | Lower for bulk ops |

Table 8. Backup types and typical use

| Type | When to use | Restore sequence |
|---|---|---|
| Full | Regular baseline for small-to-medium DBs | Restore full |
| Differential | Between fulls to shorten restore time | Restore full, then latest differential |
| Log | For PITR under full/bulk-logged | Restore full, differentials (if any), then logs to STOPAT |

Point-in-time recovery example:
- Restore the most recent full backup WITH NORECOVERY.
- Restore the latest differential (if any) WITH NORECOVERY.
- Restore transaction log backups in sequence, using STOPAT to reach the desired timestamp.
- Recover the database and validate integrity.[^9][^24][^25]



## Performance Tuning Labs

Performance tuning in the lab is about capturing the right signals, interpreting them, and applying targeted improvements. Extended Events provide a low-overhead way to capture query-level behavior, while performance counters and DMVs frame system-wide context.[^14][^30]

Extended Events:
- Design an XE session to capture events such as sql_statement_completed and waits. Add predicates to limit noise, and set targets like event_file to persist results.
- Use SSMS to create and manage sessions, and Watch Live Data during test runs.
- Query target data via sys.fn_xe_file_target_read_file and system views to analyze results.[^14][^30]

Counters and DMVs:
- Use performance counters such as SQLServer:Databases (e.g., transaction log flushes) and AG-specific counters for replicas and database replicas to observe throughput and latency under load.[^29]
- Correlate findings with DMVs (e.g., sys.dm_exec_requests, sys.dm_os_wait_stats) to identify bottlenecks and confirm improvements after changes.

Workload patterns:
- Run targeted test queries that exhibit join strategies, scans versus seeks, or parameter sniffing issues. Capture before/after plans and metrics.

Table 9. XE session design patterns

| Event | Predicate | Target | Purpose |
|---|---|---|---|
| sql_statement_completed | sql_text like '%SELECT%HAVING%' | event_file | Measure statement durations matching a pattern |
| sql_batch_completed | database_id = <DB> | event_file | Focus on a specific database workload |
| lock_acquired | resource_description = 'KEY' | ring_buffer | Investigate locking on specific keys |
| wait_info | wait_type = 'PAGEIOLATCH_SH' | event_file | Capture I/O latch waits during heavy reads |

Table 10. Performance counters relevant to performance labs

| Counter object | Counter | Diagnostic use |
|---|---|---|
| SQLServer:Databases | Log Flushes/sec; Log Flush Write Time (ms) | Transaction log throughput and latency |
| SQLServer:Availability Replica | Bytes Received from Replica/sec; Send Queue | AG synchronization health and throughput |
| SQLServer:Database Replica | Log Bytes Received/sec; Recovery Queue | Secondary redo health and backlog |
| SQLServer:Databases | Transactions/sec; Data File Size KB | Overall database activity and size trends |

After each iteration, validate improvements against baseline metrics to confirm the impact of indexing, query changes, or configuration adjustments.[^14][^29][^30]



## Monitoring and Observability Integration

Monitoring spans from daily operator views to deep diagnostics during an incident. In this lab, you will use built-in tools first and add cloud alerts for the Azure free-tier database.

SSMS and T-SQL:
- Use the Always On dashboard for AG health; view and manage AG properties and policies.
- Inspect DMVs and catalog views to diagnose AG state, replica connectivity, and database replica states.[^7]
- Query SQL Server error logs with sp_readerrorlog for quick health checks in SSMS.[^11]

Performance Monitor:
- Track performance counters for databases and AG replicas to understand throughput, latency, and backlog.[^29]

Extended Events:
- Enable system_health and AlwaysOn_health sessions as a baseline and create targeted sessions for problem diagnosis. Use Watch Live Data for short bursts during specific tests, and limit collection with predicates and MAX_MEMORY to control resource usage.[^14][^15][^30]

Azure Monitor for the free-tier database:
- In the Azure portal, configure alert rules on the “Free amount remaining” metric to avoid unexpected pauses or charges. Use the Metrics page to visualize remaining capacity and trends.[^2]

Table 11. Monitoring tools and common use cases

| Tool | Use case |
|---|---|
| SSMS dashboard (Always On) | Visualize AG health and synchronization |
| T-SQL (DMVs/catalog) | Inspect replica states, endpoints, and listener |
| Performance Monitor | Track counters for databases and AG replicas |
| Extended Events | Diagnose workload, waits, and long-running operations |
| Azure Monitor (portal) | Alerts and metrics for free-tier usage |

Good monitoring is targeted: collect the minimum data that answers the question, retain it for long enough to confirm trends, and avoid letting telemetry become noise.[^7][^14][^29][^30]



## Security Audit Scenarios

Security exercises in the lab cultivate a defense-in-depth mindset. Rather than a checklist, we will run scripts to inventory accounts, roles, and permissions; inspect encryption; examine auditing; and highlight dangerous surface area such as xp_cmdshell. We also reinforce patching hygiene and safe query coding practices.

Security posture auditing:
- Inventory server principals and database users to identify unexpected accounts.
- Review server and database role memberships, focusing on sysadmin and securityadmin roles.
- Inspect explicit permissions granted outside of roles, which can erode least-privilege guarantees.
- Check database encryption state, traffic encryption (connections), and backup encryption status.
- Look for active audits and catalog common attack surface extenders (linked servers, HADR states, mirroring, replication, external data sources).[^\[10\]]

Dangerous configurations:
- xp_cmdshell can bypass application-layer protections; verify it is disabled unless absolutely necessary and controlled.[^10]
- Review patching status via @@Version for both SQL Server and the OS; maintain regular patching cadence.[^10]

SQL injection and safe coding:
- Demonstrate parameterization and robust error handling as a contrast to unsafe dynamic SQL patterns. Emphasize consistent parameter usage and limiting information returned to callers.

Community tool:
- Consider using the community sp_checksecurity tool for a broader, automated assessment with guided checks.[^31]

Table 12. Security audit checklist mapping

| Objective | Relevant objects | Validation output |
|---|---|---|
| Inventory server logins | sys.server_principals | Account list by type |
| Inventory server roles | sys.server_role_members | Role membership roster |
| Review explicit server perms | sys.server_permissions | Non-role-based grants |
| Inventory database users | sys.database_principals; role_members | User list and roles |
| Review database perms | sys.database_permissions | Explicit grants and exceptions |
| Check encryption state | sys.dm_database_encryption_keys; sys.certificates | Encrypted status, key info |
| Check traffic encryption | sys.dm_exec_connections | Encrypted session counts |
| Check backup encryption | msdb backup sets/media families | Encryption in backup history |
| Audit settings | sys.server_audits | Server audit configurations |
| High-risk features | sys.configurations (xp_cmdshell) | Enabled/disabled |
| Patch level | @@Version | Version and patch level |

The outcome is an actionable report that identifies specific remediation steps and strengthens the security posture in line with least privilege and defense-in-depth.[^10][^31]



## Troubleshooting Playbooks

Robust operators rely on playbooks. These quick-reference guides reduce downtime by accelerating diagnostics and guiding corrective actions during known failure modes.

Table 13. Troubleshooting matrix

| Problem | Likely cause | Diagnostics | Remediation |
|---|---|---|---|
| Setup warnings or feature failures | OS/feature prerequisites unmet; sector size issues; firewall blocks | Review Setup logs; check hardware/software requirements; verify firewall rules | Update OS/requirements; adjust registry for sector sizes; open required ports[^1][^5] |
| Container fails to start or accept connections | EULA not accepted; weak SA password; missing port mapping; image tag mismatch | docker logs/logs inside container; check errorlog for “ready for client connections” | Accept EULA; set MSSQL_SA_PASSWORD; correct -p mapping; use supported tags and x86-64 host[^3] |
| AG join failures or replica not joining | Endpoint misconfigured; WSFC not healthy; firewall blocks | sys.database_mirroring_endpoints; WSFC cluster state; listener/DNS issues | Recreate endpoint with correct auth; fix cluster/quorum; open ports; ensure paths and file movement are valid[^7] |
| Failover does not behave as expected | Misconfigured failover mode; quorum loss; network partitions | AG properties; WSFC cluster events; health policies | Align failover modes with HA/DR intent; restore quorum; adjust flexible failover policy[^7] |
| Cannot connect through listener | DNS/VIP not configured; client network settings | nslookup; telnet to listener port; connection string usage | Create/modify listener; ensure client uses listener DNS name; verify subnets and VIPs[^7] |

In all cases, start with the simplest checks: logs and explicit system state. SSMS “Tips and tricks” are also helpful when diagnosing common connection issues, such as verifying instance names and locating error log files without a full connection.[^3][^7][^11]



## Lab Exercises and Complexity Levels

The following exercise set scaffolds skills from foundational to advanced, reinforcing theory with muscle memory and iterative validation.

Beginner (single instance):
- Install SQL Server (Windows) or run SQL Server in Docker; connect via SSMS; explore databases, tables, and query plans. Create a small test database and run a basic backup/restore. Familiarize with sp_readerrorlog and firewall rules for connectivity.[^5][^3][^11]
- Extended Events basics: create a simple session that captures sql_statement_completed with a filter, and interpret target data using SSMS and sys.fn_xe_file_target_read_file.[^14]

Intermediate (backups and refresh):
- Full/differential/log backup strategies: implement a backup schedule and test restore chains, including RESTORE VERIFYONLY and RESTORE WITH STOPAT for PITR. Validate DBCC CHECKDB and measure restore times for the lab’s target RTO.[^9][^24][^25]
- Database refresh automation: implement a PowerShell script that copies a copy-only full backup and restores it, adding log restores where needed. Schedule with SQL Agent or Windows Scheduler. Include validation steps.[^20][^21]

Advanced (AG, performance, monitoring, security):
- Always On AG setup: build WSFC, enable Always On, create endpoints, provision AG with a listener, configure read-only routing, and execute planned and forced failovers. Validate with AG dashboard and counters.[^7]
- Performance tuning: design XE sessions to capture problem queries and waits; analyze impact via Performance Monitor counters (e.g., Log Flush metrics) and confirm improvements after indexing and plan adjustments.[^14][^29][^30]
- Monitoring integration: keep system_health and AlwaysOn_health sessions enabled; for the Azure free-tier database, configure Azure Monitor alerts on free usage.[^2][^7][^14]
- Security audit: run T-SQL audit scripts to inventory logins, roles, permissions, encryption, audits, and dangerous configurations (e.g., xp_cmdshell). Document deviations and remediations.[^10][^31]

For learners using Docker, integrate persistence best practices from the beginning to avoid data loss during environment reset. For those using Azure free tier, emphasize monitoring and alerting to make the most of limited compute/storage while learning PaaS operations.[^2][^3][^18]



## Maintenance and Best Practices

Sustained learning requires a maintained lab. Establish a cadence for updates, refreshes, backups, and monitoring adjustments, and document everything.

Patch cadence:
- Apply SQL Server servicing updates promptly, and keep the OS patched. Slipstream updates where possible to reduce setup variance in the lab. Confirm patch level with @@Version in both on-prem and PaaS contexts (as applicable).[^\[5\]][^1]

Backup retention:
- Define retention for full/differential/log backups and verify them periodically with RESTORE VERIFYONLY. Track space usage and file integrity via checksums. Keep backup inventory and restore runbooks current.[^9][^25]

Monitoring adjustments:
- Fine-tune XE session filters and memory to reduce noise and disk usage. Keep system health sessions enabled and rotate target files safely.
- Review Performance Monitor counters for growth trends and AG health, adjusting sampling intervals to balance clarity and overhead.[^14][^29]

Documentation and runbooks:
- Maintain a simple operations manual: backup locations, restore steps, AG listener details, alert thresholds, and troubleshooting playbooks. Include a history of changes and validation results for each lab iteration.[^9][^7]

Table 14. Lab maintenance schedule (indicative)

| Frequency | Task |
|---|---|
| Weekly | Review error logs; validate backup success and size trends; spot-check XE target usage |
| Monthly | Apply patches/updates; validate installation; rotate test credentials; refresh sample data |
| Quarterly | Rehearse full DR restore; validate RTO/RPO; audit security posture; tune monitoring thresholds |



## Appendices and Templates

A. SSMS keyboard shortcuts and productivity tips
- Use SSMS tricks for faster navigation and T-SQL authoring: comment/uncomment (Ctrl+K, Ctrl+C / Ctrl+K, Ctrl+U), indent controls (Tab/Shift+Tab), filtering in Object Explorer, and quick access to the error log and instance properties.[^11]

B. Command templates

Docker (bash/powershell) for lab reset:
- Start a container (example values; replace password and tag):
  - docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=LabStrong!Passw0rd" -p 1433:1433 --name sql1 --hostname sql1 -d mcr.microsoft.com/mssql/server:2022-latest
- Inspect:
  - docker ps -a
  - docker exec -it sql1 cat /var/opt/mssql/log/errorlog | grep "ready for client connections"
- Stop/remove (and re-run with different parameters to reset):
  - docker stop sql1 && docker rm sql1

XE (T-SQL) session creation example:
- CREATE EVENT SESSION [LabXE] ON SERVER
  ADD EVENT sqlserver.sql_statement_completed
  (ACTION(sqlserver.sql_text) WHERE ([sqlserver].[like_i_sql_unicode_string]([sqlserver].[sql_text], N'%SELECT%HAVING%')))
  ADD TARGET package0.event_file(SET filename=N'C:\Temp\LabXE_Target.xel');
- ALTER EVENT SESSION [LabXE] ON SERVER STATE = START;

Backup and restore (T-SQL snippets):
- Full backup:
  - BACKUP DATABASE [LabDB] TO DISK = N'C:\Backups\LabDB_Full.bak' WITH CHECKSUM;
- Differential backup:
  - BACKUP DATABASE [LabDB] TO DISK = N'C:\Backups\LabDB_Diff.bak' WITH CHECKSUM;
- Transaction log backup:
  - BACKUP LOG [LabDB] TO DISK = N'C:\Backups\LabDB_Log.trn' WITH CHECKSUM;
- Restore with PITR (illustrative):
  - RESTORE DATABASE [LabDB] FROM DISK = N'C:\Backups\LabDB_Full.bak' WITH NORECOVERY, REPLACE;
  - RESTORE DATABASE [LabDB] FROM DISK = N'C:\Backups\LabDB_Diff.bak' WITH NORECOVERY;
  - RESTORE LOG [LabDB] FROM DISK = N'C:\Backups\LabDB_Log.trn' WITH STOPAT = '2025-11-25 10:00:00', RECOVERY;
- Verify backup:
  - RESTORE VERIFYONLY FROM DISK = N'C:\Backups\LabDB_Full.bak';

C. Sample outputs and baseline metrics
- XE target data: Provide sample queries to read .xel files and summarize durations.
- Performance counters: Baselines for Log Flushes/sec under normal and loaded conditions; track changes after tuning.[^14][^29]

D. Azure Monitor alert examples (free-tier usage)
- Metric: Free amount remaining (unit: vCore seconds)
- Threshold: Less than 10,000 vCore seconds
- Action: Send email or other notification to prompt disconnecting idle clients and avoid hitting the cap prematurely.[^2]

E. Community tools
- sp_checksecurity for expanded security audits.[^31]



## References

[^1]: SQL Server 2022: Hardware & software requirements - Microsoft Learn. https://learn.microsoft.com/en-us/sql/sql-server/install/hardware-and-software-requirements-for-installing-sql-server-2022?view=sql-server-ver17  
[^2]: Deploy Azure SQL Database for free (Free tier offer) - Microsoft Learn. https://learn.microsoft.com/en-us/azure/azure-sql/database/free-offer?view=azuresql  
[^3]: Run SQL Server on Linux in Docker - Quickstart - Microsoft Learn. https://learn.microsoft.com/en-us/sql/linux/quickstart-install-connect-docker?view=sql-server-ver17  
[^4]: Azure SQL Database - Product page. https://azure.microsoft.com/en-us/products/azure-sql/database  
[^5]: SQL Server Installation Guide (Windows) - Microsoft Learn. https://learn.microsoft.com/en-us/sql/database-engine/install-windows/install-sql-server?view=sql-server-ver17  
[^6]: Manually configure an Always On availability group for SQL Server on Azure VMs (single subnet) - Microsoft Learn. https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/availability-group-manually-configure-tutorial-single-subnet?view=azuresql  
[^7]: Getting started with Always On Availability Groups - Microsoft Learn. https://learn.microsoft.com/en-us/sql/database-engine/availability-groups/windows/getting-started-with-always-on-availability-groups-sql-server?view=sql-server-ver17  
[^8]: Configure SQL Server Always On availability groups with Windows Server Failover Clustering - Google Cloud. https://docs.cloud.google.com/compute/docs/instances/sql-server/configure-availability  
[^9]: Back up and restore of SQL Server databases - Microsoft Learn. https://learn.microsoft.com/en-us/sql/relational-databases/backup-restore/back-up-and-restore-of-sql-server-databases?view=sql-server-ver17  
[^10]: A Simple SQL Server Security Checklist - Straight Path Solutions. https://straightpathsql.com/archives/2022/07/a-simple-database-security-checklist/  
[^11]: Tips and tricks for using SQL Server Management Studio (SSMS) - Microsoft Learn. https://learn.microsoft.com/en-us/ssms/tutorials/ssms-tricks  
[^12]: Back up and restore a SQL Server database with SSMS - Quickstart - Microsoft Learn. https://learn.microsoft.com/en-us/sql/relational-databases/backup-restore/quickstart-backup-restore-database?view=sql-server-ver17  
[^13]: BACKUP (Transact-SQL) - Microsoft Learn. https://learn.microsoft.com/en-us/sql/t-sql/statements/backup-transact-sql  
[^14]: Quickstart: Extended Events in SQL Server - Microsoft Learn. https://learn.microsoft.com/en-us/sql/relational-databases/extended-events/quick-start-extended-events-in-sql-server?view=sql-server-ver17  
[^15]: Monitor system activity using Extended Events - Microsoft Learn. https://learn.microsoft.com/en-us/sql/relational-databases/extended-events/monitor-system-activity-using-extended-events?view=sql-server-ver17  
[^16]: SQL Server Extended Events tutorial - MSSQLTips. https://www.mssqltips.com/tutorial/sql-server-extended-events-tutorial/  
[^17]: Configure and customize SQL Server Linux containers - Microsoft Learn. https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-docker-container-configure?view=sql-server-ver17  
[^18]: Install Docker Engine. https://docs.docker.com/engine/installation/  
[^19]: Microsoft Container Registry: SQL Server images. https://mcr.microsoft.com/product/mssql/server/about  
[^20]: Automating a SQL Server database refresh - MSSQLTips. https://www.mssqltips.com/sqlservertip/5465/automating-a-sql-server-database-refresh/  
[^21]: Automating SQL Server database refreshes with PowerShell - SQLTabletalk. https://www.sqltabletalk.com/?p=316  
[^22]: Install SQL Server command-line tools on Linux - Microsoft Learn. https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-setup-tools?view=sql-server-ver17  
[^23]: sqlcmd utility - Microsoft Learn. https://learn.microsoft.com/en-us/sql/tools/sqlcmd/sqlcmd-utility?view=sql-server-ver17  
[^24]: RESTORE Statements (Transact-SQL) - Microsoft Learn. https://learn.microsoft.com/en-us/sql/t-sql/statements/restore-statements-transact-sql  
[^25]: RESTORE VERIFYONLY (Transact-SQL) - Microsoft Learn. https://learn.microsoft.com/en-us/sql/t-sql/statements/restore-statements-verifyonly-transact-sql  
[^26]: sp_spaceused (Transact-SQL) - Microsoft Learn. https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-spaceused-transact-sql  
[^27]: SQL Server, Availability Replica performance counters - Microsoft Learn. https://learn.microsoft.com/en-us/sql/relational-databases/performance-monitor/sql-server-availability-replica?view=sql-server-ver17  
[^28]: SQL Server, Database Replica performance counters - Microsoft Learn. https://learn.microsoft.com/en-us/sql/relational-databases/performance-monitor/sql-server-database-replica?view=sql-server-ver17  
[^29]: SQL Server, Databases performance object - Microsoft Learn. https://learn.microsoft.com/en-us/sql/relational-databases/performance-monitor/sql-server-databases-object?view=sql-server-ver17  
[^30]: Extended Events overview - Microsoft Learn. https://learn.microsoft.com/en-us/sql/relational-databases/extended-events/extended-events?view=sql-server-ver17  
[^31]: sp_checksecurity - SQL Server security assessment tool - Straight Path Solutions. https://straightpathsql.com/tool/sp_checksecurity/