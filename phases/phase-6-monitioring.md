# Phase 6 – Monitoring & Observability

## Objective

Implement proactive monitoring and automated alerting to detect infrastructure and application failures in real time. This phase validates that failures are **detected automatically**, alerts are **triggered correctly**, and the system **recovers back to a healthy state** without manual intervention.

---

## Monitoring Strategy Overview

CloudWatch was used as the primary observability platform, with alarms configured across the full request path:

- **Load Balancer health**
- **ECS service stability**
- **Compute saturation**
- **Database connectivity**

Where applicable, **Container Insights metrics** were used to provide service-level visibility.

---

## Alarm Configuration Summary

### 1. ALB – Unhealthy Targets

**Purpose:** Detect when traffic is being routed away due to failed health checks.

- **Metric:** `UnHealthyHostCount`
- **Namespace:** `AWS/ApplicationELB`
- **Threshold:** ≥ 1 unhealthy target
- **Period:** 1 minute
- **Missing data treatment:** Not breaching

**Value:** Confirms that downstream failures are detected at the edge before impacting users.

**Evidence:**  
`evidence/screenshots/phase-6/alarm-unhealthy-targets.png`

---

### 2. ECS – Running Task Count Below Desired

**Purpose:** Detect ECS task failures before recovery completes.

- **Metric:** `RunningTaskCount`
- **Namespace:** `ECS/ContainerInsights`
- **Dimensions:** ClusterName + ServiceName
- **Condition:** RunningTaskCount < DesiredTaskCount
- **Period:** 1 minute
- **Datapoints to alarm:** 1 of 1

**Key Learning:**  
Because ECS replaces failed tasks very quickly, the alarm would not naturally trigger during normal self-healing. To validate this alarm:

- ECS deployment configuration was temporarily adjusted:
  - **Minimum healthy percent set to 0**
- All running tasks were stopped
- This forced `RunningTaskCount = 0` long enough for CloudWatch to evaluate and trigger the alarm

This confirmed the alarm was correctly configured and demonstrated ECS’s fast self-healing behavior.

---

### 3. ECS – High CPU Utilization (Scaling Signal)

**Purpose:** Detect sustained load conditions requiring scaling.

- **Metric:** `ContainerCpuUtilization`
- **Namespace:** `ECS/ContainerInsights`
- **Threshold:** > 60%
- **Period:** 1 minute
- **Evaluation:** 2–3 consecutive datapoints

**Value:** Provides early warning of performance pressure before user impact occurs.

---

### 4. RDS – Database Connectivity Indicator

**Purpose:** Detect database connectivity interruptions.

- **Metric:** `DatabaseConnections`
- **Namespace:** `AWS/RDS`
- **Condition:** DatabaseConnections < 1
- **Period:** 1 minute
- **Missing data treatment:** Not breaching

**Validation Method:**  
The alarm was validated by intentionally stopping the RDS instance, which caused database connections to drop to zero and triggered the alarm. Upon restarting the database, the alarm returned to OK.

## Alarm Notification Flow

All alarms publish to a centralized SNS topic:

- **SNS Topic:** `phase6-monitoring-alerts`
- **Delivery:** Email notifications
- **States monitored:** ALARM and OK transitions

This ensures both **failure detection** and **recovery confirmation** are observable.

---

## Validation Testing (Phase 6C)

To prove alarms worked as intended, controlled failures were introduced:

| Action Performed     | Alarm Triggered            |
| -------------------- | -------------------------- |
| ECS task termination | ECS RunningTaskCount alarm |
| Health check failure | ALB Unhealthy Target alarm |
| Sustained load       | ECS High CPU alarm         |
| Database restart     | RDS Connectivity alarm     |

Each alarm was observed transitioning:

- **OK → ALARM**
- **ALARM → OK**

Screenshots were captured for both states to provide clear evidence of detection and recovery.

---

## Key Takeaways

- Container Insights provides deeper, service-level visibility than classic ECS metrics.
- ECS self-healing is fast enough that alarms may require deliberate testing strategies to validate.
- Monitoring must account for **both failure detection and recovery confirmation**.
- Proper observability validates not just architecture, but **operational readiness**.

---

## Outcome

Phase 6 confirms that the infrastructure can:

- Detect failures automatically
- Alert operators without manual inspection
- Recover to a healthy state
- Provide measurable evidence of resilience and observability

This completes the monitoring and alerting implementation for the project.
