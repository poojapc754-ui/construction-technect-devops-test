# Troubleshooting: "Unable to Connect to Database" — Construction Technect

## Scenario
- Application was working yesterday, broken today
- Error: "Unable to connect to database"
- EC2/application server: running
- PostgreSQL database: running
- Users can reach the application (frontend loads)
- Backend cannot establish a database connection
- Confirmed target: PostgreSQL on port 5432

## Troubleshooting Methodology

I approach this in order from **fastest to check → most likely cause**, so I don't waste time on complex network debugging before ruling out something simple like a config typo.

---

 Step 1: Check Application Logs First
**Why first:** Logs usually tell you the *exact* error (timeout vs. authentication failure vs. DNS failure), which immediately narrows down which of the categories below to investigate. This takes 30 seconds and often solves 50% of the guesswork.

**What I'd check:**
- CloudWatch Logs for the ECS/EC2 application
- Look for the exact PostgreSQL error message, e.g.:
  - `ECONNREFUSED` → network/port/security group issue
  - `ETIMEDOUT` → network reachability issue
  - `password authentication failed` → credentials issue
  - `no pg_hba.conf entry` → database-side access rule issue
    
    Step 2: Verify Environment Variables / Application Configuration
Why: The most common real-world cause of "worked yesterday, broken today" is a changed or expired environment variable — especially if RDS was recently modified, failed over, or credentials were rotated.
What I'd check:
DB_HOST, DB_PORT, DB_USER, DB_PASSWORD, DB_NAME values in the running container/instance
Whether these are pulled from Secrets Manager correctly (not stale/cached)
Command/Tool:
# If on EC2, check environment directly
printenv | grep DB_

# If on ECS, check the task definition's current values
aws ecs describe-task-definition --task-definitio

  - `getaddrinfo ENOTFOUND` → DNS/endpoint issue

**Command/Tool:**
```bash
aws logs tail /ecs/construction-technect-backend --follow

Step 3: Verify the Database Endpoint
Why: RDS endpoints can change after a failover (Multi-AZ), a restore, or if someone pointed the app at the wrong instance.
What I'd check:
The actual current RDS endpoint from the AWS Console/CLI
Compare it to what's configured in the app's environment variables
Command/Tool:
aws rds describe-db-instances --db-instance-identifier construction-technect-db \
  --query "DBInstances[0].Endpoint"
Step 4: Verify Database Credentials
Why: If Secrets Manager rotated the password automatically but the app is using a cached/old value, this is a classic silent failure.
What I'd check:
Try connecting manually with the credentials the app is using
Confirm Secrets Manager has the current password and the app is actually pulling from it (not an old env var)
Command/Tool:
psql -h <db-endpoint> -U <db-user> -p 5432 -d <db-name>
If this fails with "password authentication failed" → credentials confirmed as the issue.
If this succeeds manually but the app still fails → the issue is app-side (stale config), not database-side.
Step 5: Check Network Connectivity (Port 5432)
Why: Since we already know the target is port 5432, I'd test whether the app server can even reach that port on the database — this isolates network-layer issues from everything else.
What I'd check:
Can the app server reach the DB host on port 5432 at all?
Command/Tool:
telnet <db-endpoint> 5432
# or
nc -zv <db-endpoint> 5432
Connection refused/times out → network-side problem (Security Group, NACL, subnet routing)
Connection succeeds → network is fine, problem is credentials or app config
Step 6: Check Security Group Rules
Why: The single most common cause of "was working yesterday, broken today" in AWS — someone edited a Security Group rule, or the app server's IP/SG changed (e.g. new ECS task got a new SG, or Auto Scaling replaced the EC2 instance).
What I'd check:
RDS Security Group inbound rules — does it allow inbound on port 5432 from the app server's Security Group (or IP)?
Command/Tool:
aws ec2 describe-security-groups --group-ids <rds-sg-id>
Confirm there's a rule like: Type: PostgreSQL, Port: 5432, Source: <app-server-sg-id>
Step 7: Check Network ACLs and Subnet Routing
Why: Less common than Security Groups, but NACLs are stateless and can silently block traffic even if Security Groups look correct.
What I'd check:
NACL rules on both the app server's subnet and the RDS subnet
Confirm RDS is in the expected subnet group and route tables allow traffic between subnets
Command/Tool:
aws ec2 describe-network-acls --filters "Name=association.subnet-id,Values=<subnet-id>"
Step 8: Check Database Status and Health
Why: "Running" in the console doesn't always mean healthy — could be at max connections, undergoing maintenance, or storage-full.
What I'd check:
RDS status, recent events, and CloudWatch metrics (CPU, connections, storage)
Command/Tool:
aws rds describe-db-instances --db-instance-identifier construction-technect-db \
  --query "DBInstances[0].DBInstanceStatus"

aws rds describe-events --source-identifier construction-technect-db --source-type db-instance
Also check CloudWatch → DatabaseConnections metric — if it's maxed out, new connections will be refused.
How I'd Identify Application-Side vs Network-Side vs Database-Side
Test Result
Points To
psql connects manually, but app fails
Application-side (bad config, stale env var, wrong credentials in code)
telnet/nc to port 5432 times out or refuses
Network-side (Security Group, NACL, routing)
psql fails with "too many connections" or DB shows unhealthy in console
Database-side (capacity, maintenance, storage full)
psql fails with "password authentication failed"
Could be app-side (wrong password used) or database-side (password was rotated) — check
