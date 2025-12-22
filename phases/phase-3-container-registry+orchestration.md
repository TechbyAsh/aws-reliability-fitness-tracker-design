## Phase 3 – Container Registry & Orchestration (ECR + ECS/Fargate)

### Objective

Deploy the containerized backend API to AWS using managed container orchestration. This phase introduces **self-healing compute**, health-based traffic routing, and prepares the application for horizontal scaling across Availability Zones.

---

### Implementation Summary

- Created a private Amazon ECR repository to store container images.
- Pushed the backend API Docker image to ECR.
- Deployed the containerized API to Amazon ECS using the Fargate launch type.
- Configured an Application Load Balancer (ALB) with health checks to route traffic only to healthy tasks.
- Ran multiple ECS tasks concurrently to enable high availability and automated recovery.

---

### Key Design Decisions

- **Managed container runtime:** ECS on Fargate removes the need to manage EC2 instances, patching, or capacity planning.
- **Health-based routing:** An Application Load Balancer routes traffic only to tasks that pass the `/health` check.
- **Stateless orchestration:** Tasks can be safely stopped and replaced without data loss.
- **Multi-AZ readiness:** The ECS service is deployed across multiple subnets to support Availability Zone resilience.

---

### Deployment Details

- **Container registry:** Amazon ECR
- **Orchestration platform:** Amazon ECS (Fargate)
- **Service type:** ECS Service with desired task count ≥ 2
- **Load balancer:** Application Load Balancer (internet-facing)
- **Health check endpoint:** `/health`
- **Target group type:** IP (required for Fargate)

---

### Validation

- The container image is successfully stored in Amazon ECR.
- ECS tasks start and register with the target group.
- The Application Load Balancer routes traffic only to healthy tasks.
- The `/health` endpoint responds successfully when accessed through the ALB DNS endpoint.
- Multiple tasks run concurrently and respond independently.

---

### Evidence Collected

- ECR repository showing pushed container image
- Successful Docker push output
- ECS cluster overview
- ECS task definition configuration
- ECS service showing desired and running task count
- Target group health status (healthy targets)
- Browser response from ALB `/health` endpoint

---

### Phase Outcome

At the completion of Phase 3, the backend API is:

- Running on managed, serverless compute
- Automatically restarted if a task fails
- Load-balanced across multiple tasks
- Ready for auto scaling and failure testing in subsequent phases

---

### Next Phase

**Phase 4 – Database Integration & Persistence (RDS Multi-AZ)**  
The next phase introduces a resilient data layer to complement the self-healing compute tier.
