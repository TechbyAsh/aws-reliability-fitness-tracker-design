## Phase 4 – Database (RDS Multi-AZ)

### Objective

Introduce a **durable and resilient data layer** using Amazon RDS with Multi-AZ deployment. This phase ensures that application data remains available during infrastructure failures and that database access is restricted to authorized backend services only.

---

Due to AWS Free Tier limitations, the RDS instance was deployed as
Single-AZ while maintaining private subnet placement, restricted
security groups, and no public accessibility. The architecture
was intentionally designed to support Multi-AZ deployment, which
can be enabled without network redesign in a production environment.

### Implementation Summary

- Deployed an Amazon RDS database with **Single-AZ enabled** due to free tier limitations. - Placed the database in **private subnets** to prevent direct internet access. - Restricted database access using **security group rules** that allow connections only from ECS tasks. - Integrated database connectivity with the ECS/Fargate backend service.

---

### Key Design Decisions -

**single-AZ deployment:** Although Multi-AZ was not enabled due to AWS Free Tier constraints, the database was deployed in private subnets with no public access.
The subnet group spans multiple Availability Zones, allowing
**Multi-AZ** to be enabled later without architectural changes.

- **Private subnet placement:** Prevents public access to the database and reduces attack surface.
- **Security group isolation:** Database inbound traffic is restricted to the ECS task security group only.
- **Managed database service:** RDS handles backups, patching, and failover, reducing operational overhead.

  ***

### Network & Security Configuration

- **VPC:** Application VPC
- **DB Subnet Group:** Includes only private subnets across multiple AZs
- **Public accessibility:** Disabled
- **Database security group:**
- Inbound: Database port (e.g. 3306) from ECS task security group
- Outbound: Default

---

### Validation

- The RDS instance is deployed with **single-AZ enabled**.
- The database endpoint is reachable from ECS tasks.
- The backend API successfully establishes a database connection.
- Application functionality is restored automatically after task redeployments.

  ***

### Evidence Collected

- RDS instance configuration showing Multi-AZ enabled
- DB subnet group containing private subnets only - Database security group inbound rule restricted to ECS tasks
- ECS task definition environment variables (redacted) - Successful database connectivity test from the backend service
- ECS service stability after database integration

  ***

### Phase Outcome

At the completion of Phase 4, the application has:

- No direct public access to the database
- Strictly controlled network access between compute and data tiers
- Improved resilience against infrastructure and Availability Zone failures

---

### Next Phase

**Phase 5 – Failure Testing & Observability**
The next phase focuses on intentionally inducing failures and using monitoring data to validate self-healing behavior across the compute and data layers.
