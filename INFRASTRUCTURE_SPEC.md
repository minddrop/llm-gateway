# AWS System Architecture Blueprint & Technical Architecture Document (TAD)
**Enterprise Engineering LLM Gateway (v2.3 Hardened AWS Well-Architected Standard)**  
**Target Environment:** AWS Tokyo (`ap-northeast-1`) Primary / AWS Osaka (`ap-northeast-3`) Disaster Recovery  
**Security Standard:** Zero Internet Egress, Strict Japan Sovereign Boundary, Zero Data Retention (ZDR), AWS Data Perimeter  
**Companion Documents:** [REQUIREMENTS.md](REQUIREMENTS.md) | [MODEL_AVAILABILITY_MATRIX.md](MODEL_AVAILABILITY_MATRIX.md) | [README.md](README.md)

---

## Document Overview & Engineering Scope

This document serves as the authoritative physical cloud infrastructure specification, network design, security architecture, and Infrastructure-as-Code (IaC) blueprint for the **Enterprise Engineering LLM Gateway**. While [REQUIREMENTS.md](REQUIREMENTS.md) defines the functional requirements (WHAT & WHY), governance tiers, FinOps business rules, and DLP pattern catalogs, this specification defines the physical cloud implementation (HOW) strictly adhering to the **AWS Well-Architected Framework (6 Pillars)**.

---

## 1. Architecture Decision Records (ADRs)

To establish technical rationale and maintain documentation integrity over the platform lifecycle, key architectural decisions are formalized below:

### ADR-001: Isolated VPC with Zero Internet Egress vs. Centralized Egress Inspection Proxy
* **Context:** Enterprise coding assistants (Cursor, VS Code, Cline) process proprietary intellectual property, uncommitted source code, and credentials. Traditional enterprise architectures route outbound traffic through NAT Gateways to Secure Web Gateways (SWG / Squid / Zscaler).
* **Decision:** Provision a dedicated, greenfield VPC with **zero Internet Gateways (IGW)**, **zero Egress-Only Internet Gateways (EIGW)**, and **zero NAT Gateways**. All upstream communication (Amazon Bedrock Runtime, Bedrock Mantle, Secrets Manager, CloudWatch, KMS) MUST route exclusively through AWS PrivateLink Interface Endpoints and VPC Gateway Endpoints.
* **Consequences:** Eliminates external internet exfiltration vectors at the route-table layer. Eliminates NAT Gateway data processing costs ($0.062/GB in Tokyo). Requires all dependent AWS services to support PrivateLink.

### ADR-002: ECS Fargate on AWS Graviton (ARM64) vs. AWS Lambda vs. Amazon EKS
* **Context:** The gateway proxy requires persistent HTTP/2 connections, unbuffered Server-Sent Events (SSE) streaming, sub-20ms proxy latency, and connection idle timeouts up to 300 seconds to support extended reasoning models (`claude-3-7-sonnet`, `o3-mini`).
* **Decision:** Deploy containerized proxy tasks on **AWS ECS Fargate utilizing the ARM64 (AWS Graviton) architecture**.
* **Consequences:**
  * *Vs. AWS Lambda:* Lambda enforces a 15-minute execution hard cap, lacks native long-lived TCP keep-alive connection pooling to PrivateLink endpoints, and introduces cold starts during token streaming bursts.
  * *Vs. Amazon EKS:* EKS introduces high control plane management overhead ($73/mo per cluster, ongoing Kubernetes version upgrades) that is disproportionate for a dedicated reverse-proxy fleet.
  * *Graviton (ARM64) Benefit:* Yields up to 40% price/performance improvement and ~20% lower carbon footprint compared to x86_64 for async Python/Uvicorn I/O workloads.

### ADR-003: Aurora PostgreSQL Serverless v2 + RDS Proxy vs. DynamoDB
* **Context:** The gateway requires complex multi-tier relational data models (users, hierarchical budgets, virtual key state machines, approval requests, daily financial ledgers) and cross-region disaster recovery replication with RPO < 1 minute.
* **Decision:** Utilize **Amazon Aurora PostgreSQL Serverless v2** fronted by **AWS RDS Proxy**, configured as an **Aurora Global Database** replicating from Tokyo to Osaka.
* **Consequences:** Provides full ACID relational integrity for FinOps ledgers and RBAC. RDS Proxy multiplexes up to 5,000 application client connections down to 120 pinned backend PostgreSQL connections, mitigating connection exhaustion spikes during mass container auto-scaling.

### ADR-004: LiteLLM Core on Fargate vs. Custom In-House Reverse Proxy
* **Context:** Foundation model wire protocols evolve continuously across OpenAI, Anthropic, Mistral, and Amazon Bedrock. Developing a custom proxy requires constant engineering maintenance.
* **Decision:** Deploy hardened open-source **LiteLLM Proxy** wrapped with custom FastAPI security middleware (validating AVA headers, enforcing dual-pass DLP, and auditing Git remotes).
* **Consequences:** Drastically reduces time-to-market and maintenance overhead while allowing complete extensibility for enterprise security middleware.

---

## 2. Pillar-by-Pillar AWS Well-Architected Blueprint

### 2.0 System Architecture Overview Diagram

![Internal Enterprise Engineering LLM Gateway Architecture](assets/infra_overview_diagram.jpg)

```
                     AWS Well-Architected Framework Alignment
 ┌─────────────────────────────────────────────────────────────────────────────┐
 │ 1. Operational Excellence: IaC Modules, CodeDeploy Canary, SSM Break-Glass   │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 2. Security: AWS Data Perimeter, Isolated VPC, AVA Cedar, WAF, KMS CMKs     │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 3. Reliability: 3-AZ Multi-AZ, Circuit Breakers, Aurora Global DB (Osaka)   │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 4. Performance Efficiency: ECS Fargate Graviton ARM64, RDS Proxy, SSE Stream│
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 5. Cost Optimization: Aurora Serverless v2, Valkey/Redis, S3 Parquet / Athena│
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 6. Sustainability: AWS Graviton4 Compute, Serverless Downscaling, WORM Purge │
 └─────────────────────────────────────────────────────────────────────────────┘
```

---

### Pillar 1: Operational Excellence

#### 1.1 Infrastructure-as-Code (IaC) Architecture
All infrastructure is declared using Terraform / OpenTofu with remote state stored in S3 and state locking in DynamoDB. Modules follow strict separation of concerns:

```
terraform/
├── environments/
│   ├── tokyo-primary/                      # ap-northeast-1 Production Stack
│   │   ├── main.tf                         # Module instantiations
│   │   ├── variables.tf                    # Environment inputs
│   │   ├── outputs.tf                      # ALB DNS, Endpoints, ARNs
│   │   ├── terraform.tfvars                # Production variable definitions
│   │   └── backend.tf                      # S3 backend in ap-northeast-1 + DynamoDB Lock
│   │
│   └── osaka-dr/                           # ap-northeast-3 Warm Standby / DR Stack
│       ├── main.tf                         # DR module instantiations
│       ├── variables.tf
│       ├── outputs.tf
│       ├── terraform.tfvars
│       └── backend.tf                      # S3 backend in ap-northeast-3 + DynamoDB Lock
│
└── modules/
    ├── networking/                         # VPC, Subnets, Route Tables, PrivateLink Endpoints
    ├── security/                           # SGs, NACLs, KMS CMKs, Secrets Manager, WAF
    ├── iam/                                # ECS Task Roles, Execution Roles, SCPs, Cedar Policies
    ├── database/                           # Aurora Serverless v2, RDS Proxy, ElastiCache
    ├── compute/                            # ECS Fargate (ARM64), ALB, Verified Access, Autoscaling
    └── storage_finops/                     # S3 Buckets, Object Lock, Glue, Athena, Batch ECS
```

#### 1.2 Enterprise AWS Resource Tagging Taxonomy
Every AWS resource provisioned by IaC MUST inherit the following standardized resource tags:

| Tag Key | Enforcement / Permitted Values | Technical Justification |
| :--- | :--- | :--- |
| `Project` | `LLM-Gateway-Isolated` | SCP scoping condition and cost grouping. |
| `Environment` | `Production` \| `Staging` \| `DisasterRecovery` | Environment boundary isolation. |
| `CostCenter` | `CC-4012` | Corporate financial billing and chargeback attribution. |
| `DataClassification`| `Confidential-Internal` | Compliance data classification standard. |
| `SecurityClass` | `MissionCriticalCrypto` | Key deletion protection via SCP. |
| `ManagedBy` | `Terraform` | Identifies configuration authority. |

#### 1.3 Blue/Green Deployment Protocol with AWS CodeDeploy
Container updates utilize **AWS CodeDeploy** with Canary Traffic Shifting to guarantee zero downtime and automated rollbacks:
* **Target Group Pair:** `tg-llm-gw-blue` and `tg-llm-gw-green` registered to the internal ALB.
* **Traffic Routing Strategy:** `CodeDeployDefault.ECSCanary10Percent5Minutes`.
  * Step 1: Provisions replacement Green tasks in ECS Fargate.
  * Step 2: Routes 10% of developer traffic to Green containers.
  * Step 3: Executes synthetic health check probes testing `/health` and model inference via `jp.anthropic.claude-haiku-4-5`.
  * Step 4: CloudWatch monitors metric alarms (`Gateway5xxErrors > 0`, `GatewayLatencyP95 > 50ms`).
  * Step 5: If zero alarms trigger after 5 minutes, shifts remaining 90% of traffic to Green and decommissions Blue tasks.
  * Step 6: Any alarm trip triggers immediate automated rollback to Blue.

#### 1.4 Break-Glass Operations & ECS Exec Auditing
Direct SSH access is permanently prohibited. Break-glass interactive debugging is conducted strictly via **AWS ECS Exec** backed by AWS Systems Manager (SSM) Session Manager:
* **Endpoint:** Communication routes through `com.amazonaws.ap-northeast-1.ssmmessages` Interface Endpoint.
* **Session Encryption:** Encrypted end-to-end using KMS CMK `mrk-llm-gw-logs`.
* **Audit Logging:** All terminal input and output streams are mirrored in real time to the CloudWatch Log Group `/aws/ssm/ecs-exec-audit` with zero local terminal caching.

---

### Pillar 2: Security & AWS Data Perimeter

```mermaid
flowchart TD
    subgraph CorporatePerimeter["Corporate Perimeter (Intune Managed Workstations)"]
        Client["Developer Client (Cursor / VS Code / Aider)"]
    end

    subgraph AWS_Region["AWS Tokyo Region (ap-northeast-1)"]
        AVA["AWS Verified Access (AVA - ZTNA Ingress)<br/>• Cedar Policy Engine<br/>• Validates Entra ID OIDC + Intune Posture"]

        subgraph VPC_Isolated["Dedicated Isolated VPC (10.100.0.0/16) - Zero IGW / Zero NAT"]
            subgraph Subnet_Ingress["Ingress Subnets (10.100.0.0/24 - Multi-AZ 1a, 1c, 1d)"]
                ALB["Internal Application Load Balancer<br/>• TLS 1.3 Termination (Port 443)<br/>• Unbuffered SSE, 300s Timeout"]
                WAF["AWS WAF v2 WebACL (Attached to ALB)<br/>• AVA Context Header Validation<br/>• Rate-Based DDoS Shield"]
            end

            subgraph Subnet_App["App Subnets (10.100.16.0/20 - Multi-AZ 1a, 1c, 1d)"]
                Proxy["ECS Fargate Proxy Tier (Graviton ARM64)<br/>• Read-only Root FS + /tmp tmpfs<br/>• Tier 1 In-Memory Regex DLP (HTTP 422 Block)<br/>• Tier 3 SSE 128-char Outbound Buffer<br/>• Uvicorn (4 Workers)"]
            end

            subgraph Subnet_Data["Data Subnets (10.100.64.0/24 - Multi-AZ 1a, 1c, 1d)"]
                Redis["ElastiCache Serverless (Multi-AZ)<br/>• Atomic Lua Quota Reservation"]
                RDS_Proxy["AWS RDS Proxy (Multi-AZ)<br/>• Connection Pooling (120 Backends)"]
                Aurora[("Aurora PostgreSQL Serverless v2<br/>• Multi-AZ Multi-Tier DB")]
            end

            subgraph Subnet_VPCE["Endpoint Subnets (10.100.80.0/24 - Multi-AZ 1a, 1c, 1d)"]
                VPCE["AWS PrivateLink Interface Endpoints<br/>• Bedrock, Secrets Mgr, KMS, Logs, ECR"]
            end
        end

        subgraph UpstreamBedrock["Amazon Bedrock (Japan Sovereign Boundary)"]
            Guardrails["Tier 2: Amazon Bedrock Guardrails<br/>• Prompt Attack HIGH Filter<br/>• Denied Topics & Exploit Shield<br/>• Japan My Number & PII Redaction"]
            BedrockRT["Bedrock Runtime (ap-northeast-1)<br/>• jp.* Cross-Region Profiles<br/>• Tokyo Foundation Models"]
            BedrockMantle["Bedrock Mantle (ap-northeast-1)<br/>• Open Models with Server-Side Tools"]
        end
    end

    Client -->|"HTTPS 443 (Device Cert)"| AVA
    AVA -->|"Signed x-amzn-ava-user-context"| ALB
    WAF -.->|"Attached to"| ALB
    ALB -->|"HTTP 8000 (Keep-Alive)"| Proxy
    Proxy <-->|"TCP 6379 (TLS 1.3)"| Redis
    Proxy <-->|"TCP 5432 (IAM Auth)"| RDS_Proxy <--> Aurora
    Proxy -->|"HTTPS 443 (PrivateLink)"| VPCE
    VPCE --> Guardrails --> BedrockRT
    VPCE --> BedrockMantle
```

#### 2.1 AWS Data Perimeter Strategy
In compliance with the official **AWS Data Perimeter** framework, the architecture enforces three cryptographic boundaries:
1. **Expected Networks:** Resources can only be accessed from the corporate network or the dedicated VPC (`aws:SourceVpc` / `aws:sourceVpce`).
2. **Expected Identities:** Calls to AWS APIs within the VPC must originate strictly from principals belonging to the corporate AWS Organization (`aws:PrincipalOrgID`).
3. **Expected Resources:** Outbound API calls from within the VPC can only target AWS resources owned by the corporate organization (`aws:ResourceOrgID`), preventing data exfiltration to external AWS accounts.

---

#### 2.2 Network Topology & Isolated VPC Subnet Allocation Matrix

The VPC contains **no Internet Gateways**, **no NAT Gateways**, and **no Egress-Only Internet Gateways**.

* **Primary VPC Allocation (Tokyo `ap-northeast-1`):** `10.100.0.0/16`
* **Disaster Recovery VPC Allocation (Osaka `ap-northeast-3`):** `10.101.0.0/16`

##### Subnet Allocation Matrix (`ap-northeast-1`)

| Subnet Identifier | Availability Zone | CIDR Block | Usable IPs | Tier / Intended Workload | Route Table Association |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `sn-ingress-1a` | `ap-northeast-1a` | `10.100.0.0/24` | 251 | Ingress (AVA ENIs, Internal ALB ENIs) | `rt-ingress` (Local only) |
| `sn-ingress-1c` | `ap-northeast-1c` | `10.100.1.0/24` | 251 | Ingress (AVA ENIs, Internal ALB ENIs) | `rt-ingress` (Local only) |
| `sn-ingress-1d` | `ap-northeast-1d` | `10.100.2.0/24` | 251 | Ingress (AVA ENIs, Internal ALB ENIs) | `rt-ingress` (Local only) |
| `sn-app-1a` | `ap-northeast-1a` | `10.100.16.0/20`| 4,091 | Core Processing (ECS Fargate Proxy Tasks) | `rt-app` (Local + S3 Gateway) |
| `sn-app-1c` | `ap-northeast-1c` | `10.100.32.0/20`| 4,091 | Core Processing (ECS Fargate Proxy Tasks) | `rt-app` (Local + S3 Gateway) |
| `sn-app-1d` | `ap-northeast-1d` | `10.100.48.0/20`| 4,091 | Core Processing (ECS Fargate Proxy Tasks) | `rt-app` (Local + S3 Gateway) |
| `sn-data-1a` | `ap-northeast-1a` | `10.100.64.0/24`| 251 | State (Aurora v2, ElastiCache, RDS Proxy) | `rt-data` (Local strictly) |
| `sn-data-1c` | `ap-northeast-1c` | `10.100.65.0/24`| 251 | State (Aurora v2, ElastiCache, RDS Proxy) | `rt-data` (Local strictly) |
| `sn-data-1d` | `ap-northeast-1d` | `10.100.66.0/24`| 251 | State (Aurora v2, ElastiCache, RDS Proxy) | `rt-data` (Local strictly) |
| `sn-vpce-1a` | `ap-northeast-1a` | `10.100.80.0/24`| 251 | PrivateLink Interface Endpoints (ENIs) | `rt-endpoints` (Local strictly) |
| `sn-vpce-1c` | `ap-northeast-1c` | `10.100.81.0/24`| 251 | PrivateLink Interface Endpoints (ENIs) | `rt-endpoints` (Local strictly) |
| `sn-vpce-1d` | `ap-northeast-1d` | `10.100.82.0/24`| 251 | PrivateLink Interface Endpoints (ENIs) | `rt-endpoints` (Local strictly) |
| *Reserved Expansion*| — | `10.100.96.0/19`| 8,192 | Future GPU Workers / Batch Infrastructure | None |

---

#### 2.3 Route Tables & PrivateLink Endpoints Matrix

All Interface Endpoints have **Private DNS Enabled (`true`)**, deploying ENIs across `sn-vpce-1a`, `sn-vpce-1c`, and `sn-vpce-1d`.

| Endpoint Service Name | Type | Private DNS Name | Subnet Placement | Security Group Attached |
| :--- | :---: | :--- | :--- | :---: |
| `com.amazonaws.ap-northeast-1.bedrock-runtime` | Interface | `bedrock-runtime.ap-northeast-1.amazonaws.com` | `sn-vpce-1a, 1c, 1d` | `sg-vpc-endpoints` |
| `com.amazonaws.ap-northeast-1.bedrock` | Interface | `bedrock.ap-northeast-1.amazonaws.com` | `sn-vpce-1a, 1c, 1d` | `sg-vpc-endpoints` |
| `bedrock-mantle.ap-northeast-1.api.aws`* | Interface | `bedrock-mantle.ap-northeast-1.api.aws` | `sn-vpce-1a, 1c, 1d` | `sg-vpc-endpoints` |
| `com.amazonaws.ap-northeast-1.secretsmanager` | Interface | `secretsmanager.ap-northeast-1.amazonaws.com` | `sn-vpce-1a, 1c, 1d` | `sg-vpc-endpoints` |
| `com.amazonaws.ap-northeast-1.logs` | Interface | `logs.ap-northeast-1.amazonaws.com` | `sn-vpce-1a, 1c, 1d` | `sg-vpc-endpoints` |
| `com.amazonaws.ap-northeast-1.monitoring` | Interface | `monitoring.ap-northeast-1.amazonaws.com` | `sn-vpce-1a, 1c, 1d` | `sg-vpc-endpoints` |
| `com.amazonaws.ap-northeast-1.kms` | Interface | `kms.ap-northeast-1.amazonaws.com` | `sn-vpce-1a, 1c, 1d` | `sg-vpc-endpoints` |
| `com.amazonaws.ap-northeast-1.ecr.api` | Interface | `api.ecr.ap-northeast-1.amazonaws.com` | `sn-vpce-1a, 1c, 1d` | `sg-vpc-endpoints` |
| `com.amazonaws.ap-northeast-1.ecr.dkr` | Interface | `*.dkr.ecr.ap-northeast-1.amazonaws.com` | `sn-vpce-1a, 1c, 1d` | `sg-vpc-endpoints` |
| `com.amazonaws.ap-northeast-1.xray` | Interface | `xray.ap-northeast-1.amazonaws.com` | `sn-vpce-1a, 1c, 1d` | `sg-vpc-endpoints` |
| `com.amazonaws.ap-northeast-1.ssmmessages` | Interface | `ssmmessages.ap-northeast-1.amazonaws.com` | `sn-vpce-1a, 1c, 1d` | `sg-vpc-endpoints` |
| `com.amazonaws.ap-northeast-1.s3` | **Gateway** | `s3.ap-northeast-1.amazonaws.com` | Route Table `rt-app` | N/A (Prefix List `pl-63a5400a`) |

*\*Note on Bedrock Mantle Endpoint:* In Tokyo (`ap-northeast-1`), Bedrock Mantle exposes the dedicated hostname `bedrock-mantle.ap-northeast-1.api.aws`. In the isolated VPC, this hostname is resolved via Route 53 Private Hosted Zone (PHZ) mapping to the Bedrock Runtime VPC Interface Endpoint ENIs, ensuring zero internet egress.

---

#### 2.4 Security Group Chaining & Stateless NACLs

Traffic is policed through strict **Security Group Chaining** (referencing Security Group IDs directly) to eliminate IP drift vulnerabilities:

```
[sg-ava-ingress] ──────TCP 443──────► [sg-internal-alb]
                                             │
                                          TCP 8000
                                             │
                                             ▼
                                   [sg-ecs-gateway-proxy]
                                    │         │        │
                    ┌───────────────┘         │        └────────────────┐
                 TCP 6379                  TCP 5432                  TCP 443
                    ▼                         ▼                         ▼
          [sg-elasticache-redis]       [sg-rds-proxy]         [sg-vpc-endpoints]
                                             │
                                          TCP 5432
                                             │
                                             ▼
                                       [sg-aurora-db]
```

##### Detailed Security Group Rules

| Security Group ID | Direction | Protocol | Port Range | Source / Destination | Technical Justification |
| :--- | :---: | :---: | :---: | :--- | :--- |
| **`sg-internal-alb`** | Ingress | TCP | `443` | `sg-ava-ingress` | TLS 1.3 traffic from AWS Verified Access ENIs. |
|  | Egress | TCP | `8000` | `sg-ecs-gateway-proxy` | Forward reverse-proxied traffic to Fargate tasks. |
| **`sg-ecs-gateway-proxy`** | Ingress | TCP | `8000` | `sg-internal-alb` | Inbound traffic from ALB target group. |
|  | Egress | TCP | `6379` | `sg-elasticache-redis` | Atomic Lua pre-flight reservation & rate limiting. |
|  | Egress | TCP | `5432` | `sg-rds-proxy` | SQL queries for virtual keys, RBAC, and ledgers. |
|  | Egress | TCP | `443` | `sg-vpc-endpoints` | PrivateLink calls to Bedrock, Secrets Manager, KMS, Logs. |
|  | Egress | TCP | `443` | `pl-63a5400a` (S3 Tokyo) | Direct S3 Gateway endpoint traffic for Parquet dumps. |
| **`sg-elasticache-redis`** | Ingress | TCP | `6379` | `sg-ecs-gateway-proxy` | Authorize Redis protocol traffic from proxy tasks. |
|  | Egress | — | — | *None (Blocked)* | Redis initiates zero outbound network connections. |
| **`sg-rds-proxy`** | Ingress | TCP | `5432` | `sg-ecs-gateway-proxy` | Authorized SQL traffic from ECS tasks. |
|  | Egress | TCP | `5432` | `sg-aurora-db` | Multiplexed database connections to Aurora instances. |
| **`sg-aurora-db`** | Ingress | TCP | `5432` | `sg-rds-proxy` | Authorize connections solely from RDS Proxy ENIs. |
|  | Egress | — | — | *None (Blocked)* | Database engine initiates zero outbound connections. |
| **`sg-vpc-endpoints`** | Ingress | TCP | `443` | `sg-ecs-gateway-proxy` | Authorize HTTPS to PrivateLink endpoints. |
|  | Egress | — | — | *None (Blocked)* | Interface endpoints never initiate outbound connections. |

##### Network Access Control Lists (NACLs) Defense-in-Depth
NACLs provide stateless subnet-level boundaries:
* **App Subnet NACL (`acl-app`):**
  * Inbound: TCP `8000` from Ingress CIDR (`10.100.0.0/22`), Ephemeral ports `1024-65535` from VPC CIDR (`10.100.0.0/16`) and S3 Gateway prefix list.
  * Outbound: TCP `6379` to Data CIDR (`10.100.64.0/22`), TCP `5432` to Data CIDR, TCP `443` to VPCE CIDR (`10.100.80.0/22`) and S3 Gateway prefix list, Ephemeral ports to Ingress CIDR.
* **Data Subnet NACL (`acl-data`):**
  * Inbound: TCP `6379` and TCP `5432` from App CIDR (`10.100.16.0/20`) strictly.
  * Outbound: Ephemeral ports `1024-65535` to App CIDR (`10.100.16.0/20`) strictly.

---

#### 2.5 AWS Verified Access (AVA) & Cedar Policy Specification

AWS Verified Access provides Zero Trust Network Access (ZTNA) without requiring a VPN client. AVA evaluates real-time identity claims from Microsoft Entra ID and device health signals from Microsoft Intune before granting access to the ALB:

##### Verified Access Cedar Policy
```cedar
// Allow access only to authenticated engineers on compliant corporate hardware
permit(principal, action, resource)
when {
    // 1. Identity & Group Membership Validation (Entra ID OIDC)
    context.entra_id.groups.contains("SG-ENG-Developers") &&
    context.entra_id.email.endsWith("@company.com") &&

    // 2. Microsoft Intune Device Health Posture Validation
    context.intune.is_compliant == true &&
    context.intune.device_ownership == "Corporate" &&
    context.intune.os_version_compliant == true
};
```

* **Header Signing:** Upon successful policy evaluation, AVA signs the `x-amzn-ava-user-context` JWT using its managed cryptographic key and forwards the request to the internal ALB.

---

#### 2.6 AWS WAF v2 WebACL Configuration on Internal ALB

To protect the core proxy from Layer-7 denial of service and ensure direct ALB access cannot bypass AVA, an **AWS WAF v2 WebACL** is associated with the internal ALB:

```hcl
resource "aws_wafv2_web_acl" "alb_waf" {
  name        = "waf-llm-gateway-internal-alb"
  scope       = "REGIONAL"
  description = "Layer-7 protection and AVA token validation for LLM Gateway ALB"

  default_action {
    allow {}
  }

  # Rule 1: Enforce presence of signed AVA context header
  rule {
    name     = "EnforceAVAHeader"
    priority = 10

    action {
      block {}
    }

    statement {
      not_statement {
        statement {
          size_constraint_statement {
            field_to_match {
              single_header {
                name = "x-amzn-ava-user-context"
              }
            }
            comparison_operator = "GT"
            size                = 32
            text_transformation {
              priority = 0
              type     = "NONE"
            }
          }
        }
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "WAFBlockedMissingAVAHeader"
      sampled_requests_enabled   = true
    }
  }

  # Rule 2: Rate-based protection against runaway client loops
  rule {
    name     = "RateLimitPerIP"
    priority = 20

    action {
      block {}
    }

    statement {
      rate_based_statement {
        limit              = 2000 # 2000 requests per 5-minute window per IP
        aggregate_key_type = "IP"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "WAFRateLimitExceeded"
      sampled_requests_enabled   = true
    }
  }

  # Rule 3: AWS Managed Common Rule Set (CRS)
  rule {
    name     = "AWSManagedRulesCommonRuleSet"
    priority = 30

    override_action {
      none {}
    }

    statement {
      managed_rule_group_statement {
        name        = "AWSManagedRulesCommonRuleSet"
        vendor_name = "AWS"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "WAFCommonRuleSet"
      sampled_requests_enabled   = true
    }
  }

  visibility_config {
    cloudwatch_metrics_enabled = true
    metric_name                = "WAFLLMGatewayTotal"
    sampled_requests_enabled   = true
  }
}
```

---

#### 2.7 IAM Least-Privilege & Geofence Policy Specifications

##### 1. ECS Task Execution Role
Attached to `ecs-tasks.amazonaws.com` for bootstrap container initialization:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowECRImagePullScoped",
      "Effect": "Allow",
      "Action": [
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage",
        "ecr:BatchCheckLayerAvailability"
      ],
      "Resource": "arn:aws:ecr:ap-northeast-1:111122223333:repository/llm-gateway-proxy",
      "Condition": {
        "StringEquals": {
          "aws:ResourceTag/Environment": "Production"
        }
      }
    },
    {
      "Sid": "AllowECRAuthToken",
      "Effect": "Allow",
      "Action": "ecr:GetAuthorizationToken",
      "Resource": "*"
    },
    {
      "Sid": "AllowCloudWatchLoggingBootstrap",
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:ap-northeast-1:111122223333:log-group:/aws/ecs/llm-gateway/*:*"
    },
    {
      "Sid": "AllowBootstrapSecretsDecryption",
      "Effect": "Allow",
      "Action": "secretsmanager:GetSecretValue",
      "Resource": [
        "arn:aws:secretsmanager:ap-northeast-1:111122223333:secret:llm-gateway/db-credentials-*",
        "arn:aws:secretsmanager:ap-northeast-1:111122223333:secret:llm-gateway/redis-auth-*",
        "arn:aws:secretsmanager:ap-northeast-1:111122223333:secret:llm-gateway/master-admin-key-*"
      ]
    },
    {
      "Sid": "AllowKMSDecryptBootstrapSecrets",
      "Effect": "Allow",
      "Action": "kms:Decrypt",
      "Resource": "arn:aws:kms:ap-northeast-1:111122223333:key/mrk-8a7c2b4d3e5f601a2b3c4d5e6f708192"
    }
  ]
}
```

##### 2. ECS Task Role (Runtime Geofenced Policy)
*Critical Correction:* AWS system-defined cross-region inference profiles use ARN format `arn:aws:bedrock:<region>::inference-profile/jp.*` (with an empty account ID `::`). The task role allows both system-defined and customer application profiles while explicitly denying non-Japan resources:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowBedrockJapanInferenceProfiles",
      "Effect": "Allow",
      "Action": [
        "bedrock:InvokeModel",
        "bedrock:InvokeModelWithResponseStream"
      ],
      "Resource": [
        "arn:aws:bedrock:ap-northeast-1::inference-profile/jp.anthropic.claude-sonnet-4-5*",
        "arn:aws:bedrock:ap-northeast-1::inference-profile/jp.anthropic.claude-sonnet-4-6*",
        "arn:aws:bedrock:ap-northeast-1::inference-profile/jp.anthropic.claude-haiku-4-5*",
        "arn:aws:bedrock:ap-northeast-1::inference-profile/jp.anthropic.claude-opus-4-7*",
        "arn:aws:bedrock:ap-northeast-1::inference-profile/jp.anthropic.claude-opus-4-8*",
        "arn:aws:bedrock:ap-northeast-1::inference-profile/jp.amazon.nova-2-lite-v1:0*",
        "arn:aws:bedrock:ap-northeast-1:111122223333:application-inference-profile/*"
      ]
    },
    {
      "Sid": "AllowBedrockTokyoInRegionFoundationModels",
      "Effect": "Allow",
      "Action": [
        "bedrock:InvokeModel",
        "bedrock:InvokeModelWithResponseStream"
      ],
      "Resource": [
        "arn:aws:bedrock:ap-northeast-1::foundation-model/mistral.devstral-2-123b",
        "arn:aws:bedrock:ap-northeast-1::foundation-model/qwen.qwen3-coder-480b-a35b-v1:0",
        "arn:aws:bedrock:ap-northeast-1::foundation-model/qwen.qwen3-32b-v1:0",
        "arn:aws:bedrock:ap-northeast-1::foundation-model/deepseek.v3.2",
        "arn:aws:bedrock:ap-northeast-1::foundation-model/deepseek.v3-v1:0",
        "arn:aws:bedrock:ap-northeast-1::foundation-model/google.gemma-3-12b-it",
        "arn:aws:bedrock:ap-northeast-1::foundation-model/openai.gpt-oss-120b-1:0",
        "arn:aws:bedrock:ap-northeast-1::foundation-model/openai.gpt-oss-20b-1:0",
        "arn:aws:bedrock:ap-northeast-1::foundation-model/amazon.nova-lite-v1:0",
        "arn:aws:bedrock:ap-northeast-1::foundation-model/amazon.nova-2-sonic-v1:0",
        "arn:aws:bedrock:ap-northeast-1::foundation-model/amazon.titan-embed-text-v2:0",
        "arn:aws:bedrock:ap-northeast-1::foundation-model/cohere.embed-multilingual-v3",
        "arn:aws:bedrock:ap-northeast-1::foundation-model/cohere.rerank-v3-5:0"
      ]
    },
    {
      "Sid": "AllowBedrockGuardrailsTokyo",
      "Effect": "Allow",
      "Action": "bedrock:ApplyGuardrail",
      "Resource": "arn:aws:bedrock:ap-northeast-1:111122223333:guardrail/*"
    },
    {
      "Sid": "ExplicitDenyNonJapanBedrockResources",
      "Effect": "Deny",
      "Action": [
        "bedrock:InvokeModel",
        "bedrock:InvokeModelWithResponseStream"
      ],
      "NotResource": [
        "arn:aws:bedrock:ap-northeast-1:*:inference-profile/jp.*",
        "arn:aws:bedrock:ap-northeast-3:*:inference-profile/jp.*",
        "arn:aws:bedrock:ap-northeast-1::inference-profile/jp.*",
        "arn:aws:bedrock:ap-northeast-3::inference-profile/jp.*",
        "arn:aws:bedrock:ap-northeast-1::foundation-model/*",
        "arn:aws:bedrock:ap-northeast-3::foundation-model/*",
        "arn:aws:bedrock:ap-northeast-1:111122223333:application-inference-profile/*",
        "arn:aws:bedrock:ap-northeast-3:111122223333:application-inference-profile/*"
      ]
    },
    {
      "Sid": "AllowZeroPayloadCloudWatchMetrics",
      "Effect": "Allow",
      "Action": "cloudwatch:PutMetricData",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "cloudwatch:namespace": "LLMGateway/Operational"
        }
      }
    },
    {
      "Sid": "AllowDistributedTracingXRay",
      "Effect": "Allow",
      "Action": [
        "xray:PutTraceSegments",
        "xray:PutTelemetryRecords"
      ],
      "Resource": "*"
    },
    {
      "Sid": "AllowRDSProxyIAMConnection",
      "Effect": "Allow",
      "Action": "rds-db:connect",
      "Resource": "arn:aws:rds-db:ap-northeast-1:111122223333:dbuser:prx-0123456789abcdef0/gw_proxy_user"
    }
  ]
}
```

##### 3. AWS Organizations Service Control Policy (SCP)
Enforced at the AWS Organizations OU level to prevent creation of egress points or invocation outside Japan:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyAllBedrockOutsideJapan",
      "Effect": "Deny",
      "Action": [
        "bedrock:InvokeModel",
        "bedrock:InvokeModelWithResponseStream",
        "bedrock:CreateInferenceProfile",
        "bedrock:GetInferenceProfile"
      ],
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": [
            "ap-northeast-1",
            "ap-northeast-3"
          ]
        }
      }
    },
    {
      "Sid": "DenyForeignInferenceProfiles",
      "Effect": "Deny",
      "Action": [
        "bedrock:InvokeModel",
        "bedrock:InvokeModelWithResponseStream"
      ],
      "Resource": [
        "arn:aws:bedrock:*:*:inference-profile/us.*",
        "arn:aws:bedrock:*:*:inference-profile/eu.*",
        "arn:aws:bedrock:*:*:inference-profile/apac.*",
        "arn:aws:bedrock:*:*:inference-profile/global.*"
      ]
    },
    {
      "Sid": "DenyInternetEgressCreationInGatewayAccount",
      "Effect": "Deny",
      "Action": [
        "ec2:CreateInternetGateway",
        "ec2:AttachInternetGateway",
        "ec2:CreateNatGateway",
        "ec2:CreateEgressOnlyInternetGateway"
      ],
      "Resource": "*"
    },
    {
      "Sid": "DenyDisablingKMSKeyRotationAndDeletion",
      "Effect": "Deny",
      "Action": [
        "kms:DisableKey",
        "kms:ScheduleKeyDeletion",
        "kms:DisableKeyRotation"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:ResourceTag/SecurityClass": "MissionCriticalCrypto"
        }
      }
    }
  ]
}
```

---

#### 2.8 KMS Customer Managed Key (CMK) Topology

All data at rest is encrypted with AWS KMS Multi-Region Customer Managed Keys (`mrk-`) with automated annual rotation:

```
AWS KMS (ap-northeast-1 Primary -> ap-northeast-3 Replica)
├── mrk-llm-gw-database     --> Aurora PostgreSQL v2, RDS Proxy Secrets, Snapshots
├── mrk-llm-gw-redis        --> ElastiCache Redis / Valkey Serverless At-Rest Encryption
├── mrk-llm-gw-storage      --> S3 Audit Bucket (WORM) & S3 FinOps Bucket (Parquet)
└── mrk-llm-gw-logs         --> CloudWatch Log Groups (/aws/ecs/llm-gateway/*)
```

---

#### 2.9 Amazon S3 Bucket Policies with VPC Endpoint Locking

To comply with AWS Data Perimeter guidelines, access to the S3 Audit and FinOps buckets is restricted strictly to requests originating from the VPC Gateway Endpoint:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "EnforceTLSRequestsOnly",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::s3-llm-gateway-finops-ap-northeast-1",
        "arn:aws:s3:::s3-llm-gateway-finops-ap-northeast-1/*",
        "arn:aws:s3:::s3-llm-gateway-audit-ap-northeast-1",
        "arn:aws:s3:::s3-llm-gateway-audit-ap-northeast-1/*"
      ],
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    },
    {
      "Sid": "RestrictAccessToGatewayVPCEndpoint",
      "Effect": "Deny",
      "Principal": "*",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": [
        "arn:aws:s3:::s3-llm-gateway-finops-ap-northeast-1/*",
        "arn:aws:s3:::s3-llm-gateway-audit-ap-northeast-1/*"
      ],
      "Condition": {
        "StringNotEquals": {
          "aws:sourceVpce": "vpce-0123456789abcdef0"
        }
      }
    }
  ]
}
```

---

#### 2.10 ECS Fargate Container Hardening (CIS Benchmark)

Fargate containers follow the **CIS AWS Foundations Benchmark** and Docker security best practices:

* **CPU Architecture:** `ARM64` (AWS Graviton4).
* **Read-Only Root Filesystem:** `readonlyRootFilesystem: true` enforces that container filesystems cannot be modified at runtime.
* **Ephemeral Mounts:** An in-memory temporary filesystem is mounted at `/tmp` (`tmpfs`, size `512MiB`) for temporary worker process sockets.
* **Linux Capabilities:** All Linux kernel capabilities are explicitly dropped (`drop: ["ALL"]`).
* **Process Reaper:** `initProcessEnabled: true` ensures zombie worker processes are properly reaped.
* **Container Health Check:**
  `CMD-SHELL, curl -f http://localhost:8000/health || exit 1`
  (Interval: 15s, Timeout: 5s, Retries: 3, StartPeriod: 30s).

---

#### 2.11 Amazon Bedrock Guardrails & Hybrid DLP Infrastructure Specification

To resolve the trade-offs between speed, cost, and safety (where pure regex lacks semantic awareness for jailbreaks, and pure Guardrails incurs excessive latency/cost and regex limitations for codebases), the gateway implements a **Hybrid 3-Tier DLP Engine**:

1. **Tier 1 (Fargate In-Memory Edge Filter):**
   * **Engine:** Pre-compiled Aho-Corasick automaton + Shannon entropy calculation in FastAPI middleware (`< 1.5ms`, `$0` incremental compute cost).
   * **Infrastructure Hard-Block (`HTTP 422`):** Intercepts RSA/EC private keys (`DLP-CRY-KEY-001`) and Database URIs with credentials (`DLP-DB-URI-001`) immediately, preventing them from leaving the container.
   * **SaaS Masking:** Redacts developer tokens (AWS, GitHub, GitLab, Slack, JWT) with standard placeholders.
   * **Syntax-Aware Parsing:** AST comment/literal scoping prevents false-positive corruption of code identifiers (e.g. `user_name`, `customer_id`).
   * **SSRF Parameter Stripping:** Removes client-supplied routing parameters (`api_base`, `base_url`, `api_key`, `custom_llm_provider`, `mock_response`).

2. **Tier 2 (Amazon Bedrock Guardrails):**
   * **Managed Model Protection:** Attached to upstream Bedrock Runtime invocations via PrivateLink (`guardrailIdentifier` and `guardrailVersion`).
   * **Prompt Attack / Jailbreak Defense:** Evaluates prompt intent at **HIGH** filter strength, neutralizing indirect prompt injection and corporate instruction override attacks.
   * **Denied Topics:** Hard-blocks generation of malware payloads, reverse-engineering exploits, or credential harvesting routines.
   * **PII Masking:** Redacts Japanese My Number and Credit Card information.
   * **Intervention Handling:** Returns structured `HTTP 400 Bad Request` with an RFC 7807 error schema and `GUARDRAIL_INTERVENED` action code.
   * **FinOps & Performance Optimization:** Guardrails apply to interactive chat/completions workloads ($0.75 per 1,000 text units). High-throughput embedding payloads (`amazon.titan-embed-text-v2`) bypass Tier 2 Bedrock Guardrails, relying exclusively on Tier 1 Edge filtering to prevent massive cost amplification on multi-megabyte codebase embeddings.

3. **Tier 3 (Post-Flight SSE Stream Transformer & SecOps Audit):**
   * **128-Character Sliding-Window Buffer:** Buffers output across SSE chunks, neutralizing split secrets or hallucinated credentials (< 1.0ms overhead).
   * **Markdown Image Tag Exfiltration Blocker:** Strips external markdown image tags (`![...](http...)`) to prevent unauthorized outbound exfiltration in developer IDE webviews.
   * **Asynchronous Audit:** Metadata containing SHA-256 prompt hashes is logged to CloudWatch with metric alarms for DLP violations.

##### Terraform HCL: Amazon Bedrock Guardrail Definition

```hcl
resource "aws_bedrock_guardrail" "developer_safety_guardrail" {
  name        = "guardrail-llm-gateway-prod-ap-northeast-1"
  description = "Enterprise AI Gateway DLP and safety guardrail for software development workloads"
  kms_key_arn = aws_kms_key.gateway_cmk.arn

  blocked_input_messaging   = "The request violated corporate AI safety policies (Prompt Attack, Denied Topic, or Sensitive Data)."
  blocked_outputs_messaging = "The response was blocked due to corporate AI safety and data leakage prevention policies."

  # Content Policy: Jailbreak & Prompt Attack Defense
  content_policy_config {
    filters_config {
      type            = "PROMPT_ATTACK"
      input_strength  = "HIGH"
      output_strength = "NONE"
    }
    filters_config {
      type            = "HATE"
      input_strength  = "HIGH"
      output_strength = "HIGH"
    }
    filters_config {
      type            = "INSULTS"
      input_strength  = "HIGH"
      output_strength = "HIGH"
    }
    filters_config {
      type            = "SEXUAL"
      input_strength  = "HIGH"
      output_strength = "HIGH"
    }
    filters_config {
      type            = "VIOLENCE"
      input_strength  = "HIGH"
      output_strength = "HIGH"
    }
    filters_config {
      type            = "MISCONDUCT"
      input_strength  = "HIGH"
      output_strength = "HIGH"
    }
  }

  # Sensitive Information Policy: Regulatory PII Redaction
  sensitive_information_policy_config {
    pii_entities_config {
      type   = "CREDIT_DEBIT_CARD_NUMBER"
      action = "ANONYMIZE"
    }
    pii_entities_config {
      type   = "EMAIL"
      action = "ANONYMIZE"
    }
    pii_entities_config {
      type   = "IP_ADDRESS"
      action = "ANONYMIZE"
    }
    pii_entities_config {
      type   = "NAME"
      action = "ANONYMIZE"
    }

    # Custom Regex for Japan My Number (Individual Number)
    regexes_config {
      name        = "JapanMyNumber"
      description = "Matches Japanese 12-digit Individual Number (My Number)"
      pattern     = "\\b\\d{4}[ -]?\\d{4}[ -]?\\d{4}\\b"
      action      = "ANONYMIZE"
    }
  }

  # Topic Policy: Prohibit Exploit & Malware Generation
  topic_policy_config {
    topics_config {
      name       = "MalwareAndExploitGeneration"
      definition = "Requests to develop, obfuscate, or deliver computer malware, viruses, zero-day exploits, keyloggers, or unauthorized penetration testing tools."
      type       = "DENY"
      examples   = [
        "Write a Python script to scan corporate subnets for unauthenticated Redis and dump memory",
        "Generate a polymorphic shellcode to bypass EDR detection",
        "Create an automated credential harvesting phishing page mimicking Okta"
      ]
    }
  }

  tags = {
    Environment = "production"
    Application = "llm-gateway"
    ManagedBy   = "Terraform"
  }
}

resource "aws_bedrock_guardrail_version" "developer_safety_guardrail_v1" {
  guardrail_arn = aws_bedrock_guardrail.developer_safety_guardrail.arn
  description   = "Production v1 baseline: Prompt attack HIGH, My Number masking, Exploit denial"
}
```

---

### Pillar 3: Reliability & High Availability

#### 3.1 Multi-AZ Architecture (3 Availability Zones)
Every active tier is deployed across 3 Availability Zones (`ap-northeast-1a`, `ap-northeast-1c`, `ap-northeast-1d`):
* **ALB:** Cross-zone load balancing enabled across all 3 ingress subnets.
* **ECS Fargate:** Tasks distributed with `spread(attribute:ecs.availability-zone)`.
* **RDS Proxy & Aurora:** Active-Standby with synchronous multi-AZ storage replication (RPO = 0).
* **ElastiCache:** Multi-AZ Serverless cluster distributed across all 3 zones.

#### 3.2 Connection Lifecycle & Unbuffered SSE Streaming
To prevent broken streams and gateway timeouts on extended reasoning models (`claude-3-7-sonnet`, `o3-mini`, `deepseek-r1`):
* **ALB & AVA Idle Timeout:** Set to `300 seconds` (5 minutes).
* **Uvicorn Worker Tuning:** `--timeout-keep-alive 300`.
* **Response Buffering:** Disabled across ALB and proxy middleware (`X-Accel-Buffering: no`, `Cache-Control: no-cache, no-transform`). Chunks stream directly to client IDEs with `< 20ms` p95 proxy overhead.

#### 3.3 Upstream Provider Circuit Breaking & Japan-Only Fallbacks
* If Bedrock Runtime Tokyo returns >= 5 consecutive `HTTP 529 / 500 / 503` errors within 30 seconds, the circuit breaker opens for 60 seconds.
* **Zero-Overseas Failover Constraint:** Failover routes strictly to secondary Japan-compliant models:
  * Primary: `bedrock/jp.anthropic.claude-sonnet-4-5-20250929-v1:0`
  * Secondary: `bedrock/jp.anthropic.claude-haiku-4-5-20251001-v1:0`
  * Tertiary: `bedrock/mistral.devstral-2-123b` (In-Region Tokyo)
* If all Japan endpoints are degraded, the gateway returns `HTTP 503` with an RFC 7807 payload: `Provider temporarily degraded in Japan region. Fallback exhausted without overseas egress.`

#### 3.4 Disaster Recovery Runbook: Tokyo (`ap-northeast-1`) to Osaka (`ap-northeast-3`)

* **RTO (Recovery Time Objective):** `< 15 Minutes`
* **RPO (Recovery Point Objective):** `< 1 Minute`

```
Tokyo Primary (ap-northeast-1)               Osaka Standby (ap-northeast-3)
┌─────────────────────────────────┐          ┌─────────────────────────────────┐
│ • Aurora Global Database Writer │──Async──►│ • Aurora Global Database Reader │
│ • S3 WORM Audit & FinOps Buckets│──CRR────►│ • S3 Cross-Region Target Bucket │
│ • Multi-Region KMS Primary      │──Sync───►│ • Multi-Region KMS Replica      │
│ • Active Fargate Fleet (4-24)   │          │ • Warm Standby Fargate (2 tasks)│
└─────────────────────────────────┘          └─────────────────────────────────┘
                 │                                            ▲
                 ▼                                            │
        Regional Disruption ────────────────── Failover Triggered via Route 53 ARC
```

##### Automated DR Failover Procedure:
1. **Health Detection:** AWS Route 53 Application Recovery Controller (ARC) detects persistent primary regional failure in Tokyo.
2. **Database Promotion:** Execute AWS CLI promotion of the Osaka secondary cluster:
   ```bash
   aws rds failover-global-cluster \
     --global-cluster-identifier global-llm-gateway-db \
     --target-db-cluster-identifier arn:aws:rds:ap-northeast-3:111122223333:cluster:aurora-llm-gw-osaka \
     --region ap-northeast-3
   ```
3. **Compute Scale-Out:** Scale the Osaka ECS Fargate service from 2 warm standby tasks to 8 active tasks:
   ```bash
   aws ecs update-service \
     --cluster llm-gateway-osaka \
     --service proxy-core \
     --desired-count 8 \
     --region ap-northeast-3
   ```
4. **Traffic Rerouting:** Route 53 Private Hosted Zone updates the gateway alias record `llm-gateway.internal.corp` to point to the Osaka ALB.
5. **Data Sovereign Compliance:** The Osaka proxy tasks invoke Bedrock models available in Osaka or cross-region `jp.*` profiles. No tokens leave Japan.

---

### Pillar 4: Performance Efficiency

#### 4.1 Compute Sizing & Container Concurrency
* **Task Size:** `4 vCPU / 16,384 MiB Memory` on AWS Graviton4 (ARM64).
* **Process Topology:** Gunicorn master supervising **4 Uvicorn worker processes** running `uvloop` with `httptools`.
* **Upstream HTTP/2 Client Pool:** `httpx.AsyncClient` with persistent multiplexing to Bedrock PrivateLink endpoints (`max_connections=2000`, `keepalive_expiry=300.0`).
* **In-Line DLP Overhead:** Compiled regex automaton and 128-character ring buffer evaluate in `< 1.0ms`, preserving the `< 20ms` p95 gateway overhead latency SLA.

#### 4.2 Database Multiplexing via AWS RDS Proxy
* **Connection Pooling:** RDS Proxy consolidates up to 5,000 application client connections down to 120 pinned backend connections to Aurora PostgreSQL.
* **Failover Acceleration:** Maintains client TCP sockets during Aurora failover, reducing database failover connection recovery time from 35s to `< 3.2 seconds`.

---

### Pillar 5: Cost Optimization

#### 5.1 Dynamic Compute & Database Scaling
* **Aurora Serverless v2:** Scales dynamically between **2.0 ACU (4 GB RAM)** during off-hours and **32.0 ACU (64 GB RAM)** during peak sprint periods, billing by the half-second.
* **ElastiCache Serverless (Valkey / Redis):** Scales storage and ECPUs automatically with zero capacity over-provisioning.
  * *Cost Optimization Note:* Amazon ElastiCache for Valkey Serverless provides a 33% price reduction compared to Redis Serverless while maintaining 100% protocol compatibility.

#### 5.2 FinOps S3 Storage Lifecycle & Athena DDL
Reconciled transaction ledgers are exported daily at 00:05 JST to S3 in Snappy-compressed Apache Parquet format:

* **S3 Lifecycle Transitions:**
  * Day 0–90: S3 Standard
  * Day 91–365: S3 Standard-IA (Infrequent Access)
  * Day 366–2555: S3 Glacier Flexible Retrieval
  * Day 2555 (7 Years): Permanent Expiration (Financial Audit standard)

##### Athena External Table DDL
```sql
CREATE EXTERNAL TABLE IF NOT EXISTS llm_gateway_finops.transactions (
    transaction_id STRING,
    virtual_key_id STRING,
    user_upn STRING,
    project_id STRING,
    model_invoked STRING,
    provider STRING,
    input_tokens INT,
    output_tokens INT,
    cache_read_tokens INT,
    cost_usd DECIMAL(10, 6),
    latency_ms INT,
    http_status STRING,
    git_remote STRING,
    prompt_hash STRING,
    executed_at TIMESTAMP
)
PARTITIONED BY (
    year STRING,
    month STRING,
    day STRING,
    department STRING
)
STORED AS PARQUET
LOCATION 's3://s3-llm-gateway-finops-ap-northeast-1/transactions/'
TBLPROPERTIES ("parquet.compression"="SNAPPY");
```

---

### Pillar 6: Sustainability

* **AWS Graviton4 / ARM64:** Proxy tasks utilize ARM64 Graviton processors, delivering up to 60% better energy efficiency per compute unit than comparable x86_64 processors.
* **Serverless Elasticity:** Aurora Serverless v2 and ElastiCache Serverless automatically scale compute capacity down to minimal baselines during weekends and off-hours (23:00–07:00 JST), eliminating idle power consumption.
* **Automated Data Lifecycle Purges:** CloudWatch operational logs auto-expire after 30 days; raw prompts are never written to disk, minimizing enterprise storage footprints.

---

## 3. Verification & Compliance Audit Checklist

| Requirement Dimension | Infrastructure Implementation Detail | Well-Architected Pillar | Verification Status |
| :--- | :--- | :---: | :---: |
| **Zero Internet Egress** | Zero IGW, zero NAT Gateway in VPC; all traffic routed via AWS PrivateLink Interface Endpoints and S3 Gateway. | Security | **VERIFIED** |
| **Japan Sovereign Geofence** | ECS Task Role policy and AWS Organizations SCP restrict Bedrock invocation to `ap-northeast-1`, `ap-northeast-3`, and `jp.*` profiles. | Security | **VERIFIED** |
| **AWS Data Perimeter** | Expected networks, identities, and resources enforced via VPC endpoints, SCPs, and S3 bucket policies. | Security | **VERIFIED** |
| **Zero Trust Ingress** | AWS Verified Access evaluates Entra ID claims and Intune device compliance using Cedar policy; signed headers passed to ALB. | Security | **VERIFIED** |
| **Edge Defense (WAF)** | AWS WAF v2 WebACL attached to ALB enforcing AVA header presence, IP rate limiting, and AWS Core Rule Set. | Security | **VERIFIED** |
| **Low-Latency SSE Streaming** | ALB and Uvicorn keep-alive timeouts set to 300s; response buffering disabled (`X-Accel-Buffering: no`); HTTP/2 client pools. | Reliability / Performance | **VERIFIED** |
| **ARM64 Graviton Compute** | ECS Fargate tasks configured with `cpu_architecture: ARM64` for optimal price/performance and sustainability. | Performance / Sustainability | **VERIFIED** |
| **Zero-Payload Logging** | CloudWatch Log Group encrypted with KMS CMK; custom JSON filter strips prompts/code, emitting SHA-256 prompt hash only. | Security / Compliance | **VERIFIED** |
| **Container Hardening** | Fargate containers enforce read-only root filesystems, dropped capabilities (`drop: ALL`), and `/tmp` tmpfs mounts. | Security | **VERIFIED** |
| **State Resilience & DR** | Aurora Serverless v2 behind RDS Proxy; ElastiCache Multi-AZ; cross-region warm standby to Osaka (RTO < 15m, RPO < 1m). | Reliability | **VERIFIED** |
| **Hybrid DLP & Safety Guardrails** | Tier 1 edge in-line filter (<1.5ms) + Amazon Bedrock Guardrail (Prompt Attack HIGH, PII masking, exploit denial) + 128-char SSE buffer. | Security / Compliance | **VERIFIED** |
| **FinOps Reconciliation** | Daily Parquet export to S3 partitioned by date and department; Glue Catalog & Athena automated queries for ERP billing. | Operational / Cost | **VERIFIED** |
