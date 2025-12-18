# AWS Reliability Architecture – Fitness & Wellness Tracker

## Overview

This project demonstrates the design and implementation of a **highly available, self-healing backend architecture** on AWS using the **Reliability Pillar of the AWS Well-Architected Framework**.

The application is a **React Native / Expo fitness and wellness tracker** that initially existed as a frontend-only application. The goal of this project is not to build new product features, but to **design infrastructure that can automatically detect failures, recover without human intervention, and maintain availability under load or infrastructure disruption**.

---

## Architecture Goal

Design a production-ready AWS architecture that:

- Eliminates single points of failure
- Automatically replaces unhealthy resources
- Scales based on demand
- Survives instance and Availability Zone failures
- Provides measurable recovery behavior

---

## Technology Stack

### Frontend

- React Native / Expo
- Expo Web Export (`dist/`)
- Amazon S3 Static Website Hosting

### Backend

- Node.js REST API (minimal)
- Docker
- Amazon ECR
- Amazon ECS (Fargate)
- Application Load Balancer (ALB)

### Data Layer

- Amazon RDS (Multi-AZ)

### Networking & Reliability

- Multi-AZ VPC
- Public & private subnets
- Health checks
- Auto Scaling
- CloudWatch metrics & alarms

---

## Reliability Principles Applied

- Stateless compute
- Health-based traffic routing
- Automated replacement of failed resources
- Multi-AZ redundancy
- Load-based scaling
- Controlled failure testing

---

## Failure Scenarios Tested

- EC2 / container failure
- Application process crash
- Load spikes
- Database failover
- Rolling deployments

---

## Project Structure

This repository contains both **architecture documentation** and **implementation evidence**, organized by phase.

Refer to the `/phases` directory for step-by-step execution and validation.

---

## Status

🚧 In Progress – Actively building and documenting each reliability phase.

---

## Author

Ashley Johnson Jackson - TechbyAsh
AWS Solutions Architect (Portfolio Project)
