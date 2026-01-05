### Lessons Learned — Preparing /db-check for Real Resilience Testing

During Phase 5, one of the biggest insights was that a database availability test must be intentionally engineered, not assumed to “just work.” Early attempts to test DB outage recovery showed no visible failures, even while rebooting RDS — meaning the test was not accurately reflecting system behavior.

**🔍 What Went Wrong Initially**
• The original /db-check used a shared, globally initialized MySQL pool
• Because the pool was created when the app started — connections stayed alive even if the DB temporarily disappeared
• As a result, requests returned false positives (status: ok) even when the DB was unavailable
• At the same time, tying database logic into /health risked accidental cascading failures
• If /health touches the DB — a DB outage can cause ECS tasks to be marked unhealthy
• ECS may kill and replace healthy tasks unnecessarily
• The system appears broken even though only the DB is down

This taught an important principle:

Health checks MUST validate the app’s ability to serve traffic — NOT optional dependencies.

**What I Updated and Why It Mattered**

✔ /health was simplified

We intentionally kept /health fast, lightweight, and DB-independent:

     app.get('/health', (req, res) => res.status(200).json({ status: 'healthy' }))

This ensures:
• ECS & ALB always judge container health based on app availability, not DB state
• A DB outage does not cause the entire service to roll over or crash

✔ /db-check was redesigned

To ensure our outage tests were detectable, we re-engineered the `/db-check` endpoint with the following improvements:

| Feature                       | Why It Matters                                                      |
| ----------------------------- | ------------------------------------------------------------------- |
| Lazy connection pool creation | Ensures each request reflects the database’s real-time availability |
| Fresh connection per request  | Prevents cached pools from masking database failures                |
| Optional `?slow=1` query flag | Allows intentional query delay to surface outages in CloudWatch     |

This allowed the endpoint to finally return realistic errors:

{"status":"error","database":"not connected","message":"ECONNREFUSED"}

— which is what enabled meaningful resilience testing.

### Core System Design Lessons

| Lesson                                                       | Real-World Meaning                                                                   |
| ------------------------------------------------------------ | ------------------------------------------------------------------------------------ |
| Health checks should validate the service — not dependencies | Prevents cascading failures and accidental self-destruction                          |
| Test endpoints must be intentionally engineered              | Resilience testing is a feature, not a side-effect                                   |
| Graceful degradation is better than total failure            | Users can continue using the app during temporary database outages                   |
| Observability requires intentional instrumentation           | Failures must be returned clearly, slowly, and consistently for CloudWatch to detect |

---

This experience showed me that resilience is not automatic — it must be intentionally designed. I learned that app health and database availability are two separate concerns. If I had tied DB checks to the container’s health endpoint, the entire service could have been mistakenly replaced during a transient DB outage. By separating /health and /db-check, using lazy DB connections, and adding a slow-test flag, I created a system that reflects real-world behavior, supports failure testing, and avoids cascading disruption. This is a core AWS and SRE design principle — graceful degradation over catastrophic failure.
