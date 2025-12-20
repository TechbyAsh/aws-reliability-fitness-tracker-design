## Phase 2 – Backend API (Containerized)

### Objective

Introduce a minimal, stateless backend API suitable for horizontal scaling and deployment to ECS/Fargate in later phases.

---

### Implementation Summary

- Built a lightweight Node.js (Express) API designed to run as an independently deployable service.
- Added a `/health` endpoint to support load balancer and container-level health checks.
- Containerized the API using Docker to ensure consistent runtime packaging across local and cloud environments.

---

### Key Design Decisions

- **Stateless service:** The API does not write to local disk, does not maintain server-side session state, and does not rely on in-memory data persistence.
- **Port configurability:** The service binds to a configurable port using the `PORT` environment variable, ensuring compatibility with container orchestration platforms.
- **Health endpoint:** The `/health` endpoint is lightweight and dependency-free, enabling fast health validation without coupling to external services.

---

### Stateless Validation

To validate that the service is truly stateless and horizontally scalable, multiple container instances were run simultaneously on different host ports. Each instance responded independently and successfully to health check requests.

This confirmed that:

- No shared local state is required for correct operation
- Containers can be replicated without coordination
- The service can be safely scaled and replaced by an orchestrator such as ECS

---

### Validation

- The API responds successfully when run locally in a development environment.
- The Docker image builds successfully without errors.
- The container responds with a healthy status on the `/health` endpoint when run locally.
- Multiple container instances can run concurrently on different ports and return independent healthy responses.

---

### Evidence Collected

- API running locally (terminal output and health response)
- Successful Docker build output
- Container running locally with `/health` response
- Concurrent container instances returning healthy responses on different ports

---

### Phase Outcome

At the completion of Phase 2, the backend API is:

- Containerized
- Stateless
- Independently deployable
- Ready for orchestration and self-healing deployment in ECS/Fargate

---

### Next Phase

**Phase 3 – Container Registry & Orchestration (ECR + ECS/Fargate)**  
This phase introduces managed orchestration, health-based task replacement, and multi-AZ deployment.
