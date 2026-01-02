### Troubleshooting & Issue Resolution

During Phase 4 implementation, several real-world integration issues were encountered while wiring the ECS/Fargate backend to the private RDS database. Each issue was systematically diagnosed and resolved using AWS-native tooling and application-level validation.

#### Issue 1 – Database Health Endpoint Not Found

**Symptom**

- Request to `/db-check` through the Application Load Balancer returned:
  ```json
  { "error": "Route not found" }
  ```

**Diagnosis**

- Confirmed ALB routing was functional because the request reached the application.

- Error originated from the Express 404 handler, indicating the endpoint was missing at runtime.

**Resolution**

- Identified that ECS was running an older Docker image.

- Rebuilt the container image after adding the /db-check endpoint.

- Pushed the updated image to Amazon ECR.

- Updated the ECS task definition and forced a new deployment.

#### Issue 2 – Application Unable to Connect to Database

**Symptom**
{"status":"error","database":"not connected"}

**Diagnosis**

- Confirmed that the /db-check endpoint was live.

- Narrowed the issue to ECS → RDS connectivity rather than routing.

- Reviewed security groups, environment variables, and RDS configuration.

#### Issue 3 – Shared Security Group Between ALB and ECS

**Symptom**

Database connection attempts silently failed despite inbound rules appearing correct.

**Diagnosis**

- ECS tasks and the Application Load Balancer were sharing the same security group.

- This prevented RDS from correctly identifying ECS task traffic.

**Resolution**

- Created a dedicated ECS task security group.

- Updated RDS inbound rules to allow database access only from the ECS security group.

- Updated the ECS service to use the new security group and forced a redeploy.

#### Issue 4 – Database Name Misconfiguration (Root Cause)

**Symptom**

RDS configuration showed no database name defined.

ECS environment variable DB_NAME was set to the RDS instance identifier.

**Diagnosis**

Confirmed that an RDS instance can exist without an initial database schema.

Application was attempting to connect to a non-existent database.

**Resolution**

Connected directly to RDS using a database client.

Manually created the database schema.

Updated ECS environment variables to reference the correct database name.

Created a new task definition revision and redeployed the service.

#### Issue 5 – MySQL Client Installation on Amazon Linux 2023

**Symptom**

Package installation failed with:
No match for argument: mysql
Error: GPG check FAILED

**Diagnosis**

Amazon Linux 2023 no longer supports the legacy mysql package.

MySQL community client installation failed due to strict GPG validation.

**Resolution**

Installed the MariaDB client as a MySQL-compatible alternative.

Successfully connected to RDS and completed database initialization.

---

#### Final Validation

After applying all fixes:

- The /db-check endpoint returned a successful response through the ALB.

- ECS tasks successfully established private connectivity to RDS.

- Application stability was maintained across task redeployments.

---

#### Lessons Learned

- ECS runs immutable container images; application changes require image rebuilds and redeployments.

- An RDS instance identifier is not the same as a database schema.

- Security groups should be isolated per tier (ALB, ECS, RDS) to maintain clear trust boundaries.

- Runtime configuration must be verified from running ECS tasks, not assumed from task definitions.

- Amazon Linux 2023 requires updated tooling knowledge compared to earlier Amazon Linux versions.

- Simple health and database-check endpoints are invaluable for end-to-end validation.

---
