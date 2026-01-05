### Phase 5 — Failure Testing (Proof of Reliability)

**Objective**

Demonstrate that the system is resilient, self-healing, and observable under real failure scenarios. Phase 5 validates that core AWS services (ECS, ALB, Auto Scaling, and RDS) respond correctly to infrastructure stress, recover automatically, and return the system to a stable state without manual intervention.

---

### Test 1 — Kill Container (ECS Self-Healing)

**Purpose**

Prove that if a running container fails unexpectedly, Amazon ECS automatically replaces the task and restores the service to a steady state.

**Test Procedure**

- Navigated to ECS → Cluster → Service → Tasks

- Selected one running task

- Manually stopped the task to simulate a container crash

**Observations**

- The stopped task immediately transitioned to STOPPED

- ECS detected the failure and launched a replacement task

- The service returned to a stable state with the desired number of tasks running

- No user-facing downtime occurred

**Evidence**

ECS task list before termination

Task transitioning to STOPPING/STOPPED

New task launching and service stabilizing

task event logs

📸 evidence/screenshots/phase-5/test-1-kill-container/

**Outcome**

✅ PASS — ECS successfully demonstrated self-healing behavior.

---

### Test 2 — Load Spike (Service Auto Scaling)

**Purpose**

Validate that increased traffic causes the ECS service to scale out automatically and return to baseline once load subsides.

**Precondition**

ECS Service Auto Scaling enabled.

Scaling policy based on request load.

Minimum and maximum task counts configured.

**Test Procedure**

Verified auto scaling policy under ECS Service → Auto Scaling

Generated load using ApacheBench:
ab -n 5000 -c 400 http://<ALB-DNS>/health

Monitored CloudWatch dashboard during the test

**Observations**

ALB RequestCount spiked during the load test

ECS DesiredTaskCount increased in response

ECS launched additional tasks to handle traffic

After traffic ended, tasks scaled back down to baseline

Response times remained stable

**Evidence**

Auto scaling policy configuration

CloudWatch graphs showing:

RequestCount spike

DesiredTaskCount increase

Return to baseline after cooldown

📸 evidence/screenshots/phase-5/test-2-load-spike/

**Outcome**

✅ PASS — Auto scaling reacted correctly and preserved performance.

---

### Test 3A — RDS Restart (Transient Database Outage)

**Purpose**

Validate application behavior during a short-lived database interruption, simulating a transient failure such as a database reboot or maintenance event.

This test confirms that:
The application remains running during a database restart

    Database errors surface correctly

    The application automatically reconnects once the database becomes available

**Procedure**

- Step A — Baseline
  Continuously queried the /db-check endpoint to confirm normal operation

Verified database connectivity prior to restart:
while true; do
curl -s http://<ALB-DNS>/db-check
echo
sleep 2
done

📸 before-db-restart-db-check.png

- Step B — Restart RDS

Restarted the RDS instance via RDS Console → Actions → Reboot

This introduced a short database unavailability window

📸 db-reboot.png

- Step C — Observe Failure

/db-check returned database connection errors during the restart window

Errors were surfaced without impacting application health

ALB health checks remained green

ECS tasks were not replaced

📸 during-db-restart-failure.png

- Step D — Confirm Recovery

/db-check automatically returned success once RDS was available

No application restart or manual intervention required

Database connections were re-established dynamically

📸 after-db-restart-success.png

**Result**

✅ PASS — The system handled a transient database outage gracefully and recovered automatically.

---

### Test 3B — Stop & Start RDS (Intentional Full Database Outage)

**Purpose**

Demonstrate application behavior during a full database outage and confirm recovery once the database is restored.

Due to Free Tier limitations, Multi-AZ failover was unavailable. This test validates controlled outage resilience instead.

**Step C1 — Baseline**

Verified /db-check endpoint returned a successful response

Confirmed database connectivity prior to outage

📸 before-db-stop-db-check.png

**Step C2 — Stop RDS Instance**

Navigated to RDS Console → Actions → Stop

This simulated a planned database outage

**Step C3 — Observe Failure**

Repeated /db-check requests while RDS was stopped

Endpoint returned connection errors as expected

Application remained running and responsive

📸 during-db-stop-failure.png
📸 db-stop-inprogress.png
📸 db-stop-inprogress-graphs.png
📸 cloudwatch-response-drop.png
📸 alb-healthy-response.png

**Step C4 — Start RDS Instance**

Restarted the RDS instance

Waited for database to become available (5–10 minutes)

**Step C5 — Confirm Recovery**

/db-check returned success again

Database connections were restored

No ECS task replacement or cascading failures occurred

📸 after-db-start-success.png
📸cloudwatch-response-spike.png

**Key Design Validation**

/health endpoint does not depend on the database

ECS and ALB did not terminate tasks during DB outage

Application degraded gracefully and recovered automatically

**Outcome**

✅ PASS — System handled full DB outage and recovery correctly.

---

**Final Conclusion**

Phase 5 confirms that the system behaves as a production-ready, fault-tolerant architecture:

Failures are detected automatically

Recovery occurs without manual intervention

Scaling responds to real traffic demand

Dependencies fail gracefully without cascading impact

This phase demonstrates practical application of AWS reliability principles, including self-healing infrastructure, observability, and graceful degradation.
