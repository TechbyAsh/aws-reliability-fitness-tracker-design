### Troubleshooting Guide — Phase 5 (Failure Testing & Recovery)

This document captures issues encountered during Phase 5: Failure Testing and the corrective actions taken to validate system resilience. These scenarios reflect real-world AWS behavior rather than lab simulations.

### Issue 1 — Load Test Did Not Trigger ECS Auto Scaling

**Symptoms**

- ApacheBench (ab) generated traffic successfully

- ALB RequestCount spiked

- ECS RunningTaskCount and DesiredTaskCount did not increase

**Root Cause**
The ECS auto scaling policy thresholds were too conservative for the duration and intensity of the test traffic. Although traffic increased, it did not exceed the scaling alarm conditions long enough to trigger scale-out.

**Resolution**
Updated the ECS Service Auto Scaling policy:

    - Lowered target tracking threshold

    - Reduced evaluation period

- Re-ran the load test with higher concurrency
  ab -n 5000 -c 400 http://<ALB-DNS>/health

**Outcome**

- DesiredTaskCount increased during traffic spike

- ECS launched additional tasks

- Tasks scaled back down after cooldown period

**Lesson**
Auto scaling must be tuned for realistic traffic patterns, not just enabled. Validation requires adjusting thresholds to ensure scaling behavior is observable during testing.

---

### Issue 2 — Database Outage Did Not Register as Failure

**Symptoms**

RDS rebooted or stopped

/db-check endpoint continued returning success

CloudWatch metrics remained at baseline

**Root Cause**

The initial /db-check implementation reused pooled connections, allowing cached or persistent connections to survive short database interruptions.

**Resolution**

- Modified /db-check to:

  Establish a new database connection per request

  Use a reduced connection timeout

  Optionally include a delayed query (SLEEP) to surface failures during restarts

This ensured database unavailability surfaced immediately during testing.

**Outcome**

/db-check returned errors when RDS was stopped

Endpoint recovered automatically after RDS restart

ECS tasks remained stable throughout

**Lesson**

Health verification endpoints must actively test dependencies, not rely on cached connections, when used for resilience testing.

---

### Issue 3 — Health Checks Triggered ECS Task Replacement During DB Tests

**Symptoms**

ECS repeatedly replaced tasks

ALB marked targets as unhealthy

Service entered a replacement loop

**Root Cause**

Database-dependent logic was initially tied too closely to application health, causing ALB health checks to fail when the database was unavailable.

**Resolution**

Ensured /health endpoint:

Does not check database connectivity

Always returns 200 OK if the application process is running

Isolated /db-check as a diagnostic endpoint only

**Outcome**

ECS stopped replacing healthy tasks

ALB health checks remained stable

Database outages no longer caused cascading failures

**Lesson**

Health checks must validate service liveness, not dependency readiness. Dependency failures should degrade functionality, not crash infrastructure.

---

### Issue 4 — Terminal Appeared “Frozen” During Continuous Testing

**Symptoms**

Terminal appeared unresponsive when running continuous curl loops

No output displayed immediately

**Root Cause**

The continuous while true loop suppressed output formatting and did not include timestamps or newlines.

**Resolution**
Used a formatted loop with clear output:
while true; do
curl -s http://<ALB-DNS>/db-check
echo
sleep 1
done

**Outcome**

Clear visibility into success/failure transitions

Accurate evidence capture during outages

**Lesson**

When testing failure conditions, visibility matters as much as correctness.

---

### Issue 5 — Docker Build Failed Due to Incorrect Context

**Symptoms**

Docker build failed with:

'Could not read package.json'

**Root Cause**

Dockerfile assumed package.json existed at the root, but the backend lived under backend/api/src.

**Resolution**

Updated Dockerfile to copy package.json and server.js from the correct directory

Built image from the backend context

**Outcome**

Docker image built successfully

ECS deployment stabilized

**Lesson**

Docker build context must align with repository structure, especially in monorepos.

---

### Final Takeaways

- Resilience testing often exposes configuration issues, not code bugs

- Health checks must be carefully scoped

- Auto scaling requires intentional tuning

- Database failure testing requires fresh connections

- Observability is critical to validating recovery

Phase 5 confirmed the architecture behaves as a production-grade, self-healing system when properly configured.
