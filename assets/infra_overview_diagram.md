# Enterprise Engineering LLM Gateway: Infrastructure Architecture Overview

![Internal Enterprise Engineering LLM Gateway Architecture](infra_overview_diagram.jpg)

This document provides the high-level architecture overview, network topology, and Multi-AZ subnet tier mapping corresponding to the authoritative physical infrastructure specification in [INFRASTRUCTURE_SPEC.md](../INFRASTRUCTURE_SPEC.md) and functional requirements in [REQUIREMENTS.md](../REQUIREMENTS.md).

---

## 1. Network Topology & Multi-AZ Subnet Architecture

The platform is deployed in a dedicated, greenfield isolated VPC (`10.100.0.0/16`) in AWS Tokyo (`ap-northeast-1`) spanning **3 Availability Zones** (`ap-northeast-1a`, `ap-northeast-1c`, `ap-northeast-1d`) with **Zero Internet Gateways**, **Zero NAT Gateways**, and **Zero Egress-Only Gateways**.

### Subnet Allocation & Multi-AZ Placement

| Subnet Identifier | Availability Zone | CIDR Block | Usable IPs | Tier / Workload | Associated Services |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `sn-ingress-1a` | `ap-northeast-1a` | `10.100.0.0/24` | 251 | **Ingress Subnets** | AWS Verified Access (AVA) ENIs, Internal ALB ENIs |
| `sn-ingress-1c` | `ap-northeast-1c` | `10.100.1.0/24` | 251 | **Ingress Subnets** | AWS Verified Access (AVA) ENIs, Internal ALB ENIs |
| `sn-ingress-1d` | `ap-northeast-1d` | `10.100.2.0/24` | 251 | **Ingress Subnets** | AWS Verified Access (AVA) ENIs, Internal ALB ENIs |
| `sn-app-1a` | `ap-northeast-1a` | `10.100.16.0/20` | 4,091 | **App Subnets** | ECS Fargate Proxy Tasks (AWS Graviton ARM64) |
| `sn-app-1c` | `ap-northeast-1c` | `10.100.32.0/20` | 4,091 | **App Subnets** | ECS Fargate Proxy Tasks (AWS Graviton ARM64) |
| `sn-app-1d` | `ap-northeast-1d` | `10.100.48.0/20` | 4,091 | **App Subnets** | ECS Fargate Proxy Tasks (AWS Graviton ARM64) |
| `sn-data-1a` | `ap-northeast-1a` | `10.100.64.0/24` | 251 | **Data Subnets** | Aurora Serverless v2, ElastiCache Serverless, RDS Proxy |
| `sn-data-1c` | `ap-northeast-1c` | `10.100.65.0/24` | 251 | **Data Subnets** | Aurora Serverless v2, ElastiCache Serverless, RDS Proxy |
| `sn-data-1d` | `ap-northeast-1d` | `10.100.66.0/24` | 251 | **Data Subnets** | Aurora Serverless v2, ElastiCache Serverless, RDS Proxy |
| `sn-vpce-1a` | `ap-northeast-1a` | `10.100.80.0/24` | 251 | **Endpoint Subnets** | AWS PrivateLink Interface Endpoints (Bedrock, KMS, Secrets Mgr) |
| `sn-vpce-1c` | `ap-northeast-1c` | `10.100.81.0/24` | 251 | **Endpoint Subnets** | AWS PrivateLink Interface Endpoints (Bedrock, KMS, Secrets Mgr) |
| `sn-vpce-1d` | `ap-northeast-1d` | `10.100.82.0/24` | 251 | **Endpoint Subnets** | AWS PrivateLink Interface Endpoints (Bedrock, KMS, Secrets Mgr) |

---

## 2. Component Tier Breakdown & Data Flows

### Ingress & Edge Security Tier
* **Corporate Client Perimeter:** Intune-managed workstations (Cursor, VS Code, terminal CLIs) authenticate via **Microsoft Entra ID (OIDC)** and pass cryptographic device compliance claims.
* **AWS Verified Access (AVA ZTNA):** Regional edge evaluation of Cedar policies, injecting signed `x-amzn-ava-user-context` headers into the VPC.
* **Ingress Subnets:** Internal Application Load Balancer terminating TLS 1.3, protected by **AWS WAF v2 WebACL** (inspecting AVA context headers and enforcing rate limits).

### Compute & Application Tier (`App Subnets`)
* **ECS Fargate Graviton ARM64:** High-performance async proxy containers running Gunicorn with Uvicorn workers.
* **Tier 1 In-Memory Regex DLP:** In-line pre-flight scanning (<1.5ms) hard-blocking private keys and DB URIs (`HTTP 422`), redacting SaaS tokens, and stripping client SSRF parameters.
* **Tier 3 SSE Sliding-Window Buffer:** 128-character ring buffer across SSE chunks catching split credentials and stripping outbound markdown image exfiltration links (`![...](http...)`).

### Persistence & Caching Tier (`Data Subnets`)
* **ElastiCache Serverless (Redis / Valkey):** Multi-AZ low-latency state for atomic Lua quota reservations and sliding-window rate limit counters.
* **AWS RDS Proxy:** Connection pool multiplexing up to 5,000 container connections down to 120 pinned PostgreSQL sessions.
* **Aurora PostgreSQL Serverless v2:** Multi-AZ authoritative datastore scaling elastically (2.0 to 32.0 ACUs) for users, virtual key hashes, and FinOps ledger transactions.

### Upstream Model & Management Tier (`Endpoint Subnets` / PrivateLink)
* **AWS PrivateLink Interface Endpoints:** Dedicated private ENIs for zero-egress communication with AWS services.
* **Amazon Bedrock Runtime & Mantle:** Upstream Foundation Models deployed across Japan Sovereign Boundaries (`jp.*` cross-region profiles & Tokyo in-region models).
* **Tier 2: Amazon Bedrock Guardrails:** Natively integrated into **Amazon Bedrock Runtime** calls via `guardrailIdentifier` and `guardrailVersion`. Evaluates Prompt Attack defense at **HIGH** filter strength, denied topic policies, and Japan PII redaction (My Number / credit cards). There is **no separate VPC endpoint** for Guardrails; all invocations route through the `com.amazonaws.ap-northeast-1.bedrock-runtime` PrivateLink endpoint.
* **FinOps Optimization:** Interactive developer chat and coding prompts enforce Tier 2 Guardrails. High-throughput codebase embeddings (`amazon.titan-embed-text-v2`) bypass Tier 2 Guardrails to eliminate cost amplification ($0.75/1k text units).
* **Secrets & Encryption:** AWS Secrets Manager for model credentials; KMS Multi-Region Customer Managed Keys (`mrk-llm-gw`) for envelope encryption.
* **Audit & Storage:** CloudWatch Logs (zero payload, SHA-256 prompt hashes only) and S3 Parquet/WORM buckets with Object Lock.

### Disaster Recovery Tier (AWS Osaka `ap-northeast-3`)
* **Warm Standby:** Asynchronous cross-region storage replication via **Aurora Global Database** (RPO < 1 min) and S3 Cross-Region Replication (CRR).
* **Failover Readiness:** Route 53 ARC automated routing controls with pre-warmed ECS Fargate standby capacity in Osaka (RTO < 15 min).
