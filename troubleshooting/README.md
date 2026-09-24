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

### Step 1: Check Application Logs First
**Why first:** Logs usually tell you the *exact* error (timeout vs. authentication failure vs. DNS failure), which immediately narrows down which of the categories below to investigate. This takes 30 seconds and often solves 50% of the guesswork.

**What I'd check:**
- CloudWatch Logs for the ECS/EC2 application
- Look for the exact PostgreSQL error message, e.g.:
  - `ECONNREFUSED` → network/port/security group issue
  - `ETIMEDOUT` → network reachability issue
  - `password authentication failed` → credentials issue
  - `no pg_hba.conf entry` → database-side access rule issue
  - `getaddrinfo ENOTFOUND` → DNS/endpoint issue

**Command/Tool:**
```bash
aws logs tail /ecs/construction-technect-backend --follow
