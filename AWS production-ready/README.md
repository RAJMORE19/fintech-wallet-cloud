


<img width="2752" height="1536" alt="Gemini_Generated_Image_be742tbe742tbe74" src="https://github.com/user-attachments/assets/5435c6c8-7d6a-4dc2-94c4-c1b4c75278da" />



Absolutely. I’ll treat this as an **engineering apprenticeship**, not an AWS course.

The target is not “knowing AWS.” The target is being able to look at a production EKS environment and reason from **business requirement → architecture → implementation → failure → diagnosis → remediation → trade-off**.

I’m also going to deliberately delay many AWS services. For EKS, networking, IAM, Linux, containers, Kubernetes internals, and troubleshooting matter far more than knowing 100 service names.

---

# 0. The path we are going to follow

## Complete roadmap

```text
Phase 0
Linux + Networking + Engineering Foundations
        ↓
Phase 1
AWS Fundamentals + Cloud Mental Model
        ↓
Phase 2
AWS Networking Deep Dive
        ↓
Phase 3
AWS Compute + Storage + Databases + Load Balancing
        ↓
Phase 4
Containers + Docker + ECR
        ↓
Phase 5
Git + CI/CD + Terraform
        ↓
Phase 6
Kubernetes Fundamentals + Internals
        ↓
Phase 7
EKS Fundamentals
        ↓
Phase 8
Production EKS
        ↓
Phase 9
Advanced EKS + Scaling + Security + Observability
        ↓
Phase 10
Enterprise AWS / EKS Architecture
        ↓
Phase 11
Production Troubleshooting + Incident Response
        ↓
Phase 12
Senior Architecture + Interviews + Design Defense
```

There will be **overlap**, rather than rigid walls between phases. For example, Git and Terraform will start before Kubernetes, while observability and troubleshooting will begin early and become progressively deeper.

---

# 1. Phase 0 — Linux + Networking Foundations

**Goal:** Understand the operating system and network underneath everything you'll later run in AWS and Kubernetes.

### Learn deeply

**Linux**

* Filesystem
* Processes
* Users/groups
* Permissions
* Environment variables
* Services
* Signals
* stdout/stderr
* Pipes/redirection
* SSH
* Package management
* CPU/memory/disk
* `ps`, `top`, `ss`, `curl`, `grep`, `find`, `journalctl`, etc.
* Basic Bash scripting

**Networking**

* IP addresses
* CIDR
* Subnetting
* Routing
* Default gateway
* ARP concept
* TCP vs UDP
* Ports
* DNS
* HTTP/HTTPS
* TLS concept
* NAT concept
* Firewalls
* Client/server communication

### Skip for now

* Advanced Linux kernel internals
* Kernel compilation
* BGP
* Advanced routing protocols
* Deep TCP implementation
* Advanced Bash programming

### Build

A Linux-hosted application communicating with another machine.

### Break

Intentionally break:

* DNS
* Port access
* Service
* Permissions
* Routing
* Process
* Firewall

### Exit criterion

You can look at:

```text
Application
   ↓
TCP
   ↓
IP
   ↓
Route
   ↓
Network interface
   ↓
Remote machine
```

and explain what happens at every meaningful layer.

---

# 2. Phase 1 — AWS Fundamentals

**Goal:** Understand AWS's fundamental resource and security model without getting lost in hundreds of services.

### Learn

* AWS accounts
* Regions
* Availability Zones
* Resources
* ARN concept
* IAM
* EC2 fundamentals
* S3
* EBS
* CloudWatch
* AWS CLI
* AWS Console
* Basic billing/cost awareness
* Shared responsibility model

The important mental model:

```text
AWS Account
   │
   ├── Region
   │     │
   │     ├── Availability Zone
   │     │
   │     └── Resources
   │
   └── IAM identities/policies
```

### Build

A tiny application manually deployed to EC2.

### Exit criterion

You can explain:

> “What exactly exists in AWS after I launch this application?”

rather than merely clicking through the console.

---

# 3. Phase 2 — AWS Networking

This is one of the **highest-value phases in the entire roadmap**.

### Learn deeply

* VPC
* CIDR
* Public/private subnets
* Route tables
* Internet Gateway
* NAT Gateway
* Security Groups
* NACLs
* ENIs
* DNS
* VPC DNS
* VPC endpoints
* Routing
* Multi-AZ architecture
* VPC peering concept
* Transit Gateway awareness

You should eventually be able to reason about:

```text
Internet
   ↓
Internet Gateway
   ↓
Public Subnet
   ↓
Load Balancer
   ↓
Private Subnet
   ↓
Application
   ↓
Database
```

and explain **why every component exists**.

### Build

A two-tier VPC:

```text
                    Internet
                       │
                Internet Gateway
                       │
             ┌─────────┴─────────┐
             │                   │
        Public Subnet       Public Subnet
             │                   │
             └──── Load Balancer ┘
                       │
                Private Subnets
                       │
                  Application
                       │
                Database subnet
```

### Exit criterion

You can troubleshoot:

> “EC2 is running but I cannot connect.”

without randomly changing security groups.

---

# 4. Phase 3 — AWS Compute + Core Infrastructure

### Learn deeply

* EC2
* AMIs
* Instance types
* EBS
* ALB
* NLB
* Auto Scaling
* Route 53
* S3
* RDS fundamentals
* CloudWatch

### Learn enough

* ECS
* ElastiCache
* EFS
* Secrets Manager
* Systems Manager

### Awareness

* DynamoDB
* SQS
* SNS
* EventBridge
* Lambda

### Build

A real-ish application:

```text
Route 53
    ↓
ALB
    ↓
EC2 Auto Scaling
    ↓
RDS
```

with logs and metrics.

### Architecture question

Why might you choose:

```text
EC2
vs
ECS
vs
EKS
```

You will not memorize this. You'll learn the operational trade-off.

---

# 5. Phase 4 — Containers

### Learn deeply

* Docker
* Images
* Containers
* Layers
* Dockerfile
* Registries
* Networking
* Volumes
* Environment variables
* Container lifecycle
* Image tagging
* Image security
* ECR

### Build

Take the application from Phase 3:

```text
Source code
     ↓
Docker build
     ↓
Container image
     ↓
ECR
     ↓
Run container
```

### Exit criterion

You understand why:

> “The application works on my laptop but not in the container”

can happen.

---

# 6. Phase 5 — Git + CI/CD + Terraform

This is where infrastructure begins becoming **repeatable**.

### Terraform — deep

* Provider
* Resource
* Data source
* Variable
* Output
* Dependency
* Module
* State
* Remote state
* Locking
* Drift
* Import
* Plan
* Apply
* Destroy
* CI integration

Terraform is a tool for managing infrastructure, not the subject of the course.

### CI/CD — concepts first

```text
Git
 ↓
Build
 ↓
Test
 ↓
Image
 ↓
Registry
 ↓
Deploy
 ↓
Verify
 ↓
Rollback
```

We'll choose a minimal toolchain rather than learning Jenkins + GitHub Actions + GitLab CI + Argo CD + every other tool simultaneously.

### Build

Terraform creates:

```text
VPC
EC2
Security Groups
ALB
RDS
```

CI builds the container and publishes it to ECR.

---

# 7. Phase 6 — Kubernetes

This is where we stop thinking primarily in terms of AWS resources.

### Learn deeply

* Kubernetes architecture
* API server
* etcd concept
* Scheduler
* Controllers
* kubelet
* kube-proxy
* Pods
* ReplicaSets
* Deployments
* Services
* ConfigMaps
* Secrets
* Namespaces
* Labels/selectors
* Probes
* Requests/limits
* Scheduling
* Taints/tolerations
* Affinity
* StatefulSets
* DaemonSets
* Jobs/CronJobs
* Volumes
* PV/PVC
* StorageClasses
* CRDs
* Operators

The key mental model:

```text
kubectl
   ↓
API Server
   ↓
Desired State
   ↓
Controllers
   ↓
Actual State
```

You need to understand **reconciliation**, not memorize YAML.

### Build

Run Kubernetes locally.

Deploy:

```text
Application
   ↓
Deployment
   ↓
Pods
   ↓
Service
```

Then deliberately break it.

---

# 8. Phase 7 — EKS Fundamentals

Now AWS + Kubernetes finally meet.

### Learn deeply

* EKS architecture
* Control plane
* Nodes
* Managed node groups
* EKS add-ons
* VPC CNI
* CoreDNS
* kube-proxy
* EKS access
* IAM
* EKS Pod Identity
* Load balancing
* ECR integration
* EKS networking

EKS uses upstream Kubernetes and AWS's VPC CNI provides native VPC integration for pod networking; AWS specifically emphasizes understanding pod networking when operating EKS. ([AWS Documentation][1])

### Build

First EKS cluster.

But the objective is **not**:

> “I successfully ran `eksctl create cluster`.”

The objective is:

> “I understand what AWS created and how traffic, identity, scheduling and workloads move through it.”

---

# 9. Phase 8 — Production EKS

Now we start making the cluster production-like.

### Networking

* Public/private subnet architecture
* Private worker nodes
* NAT
* Load balancers
* Ingress
* DNS
* VPC CNI
* Pod IP allocation
* Security Groups
* Network Policies
* Cross-AZ traffic
* IP exhaustion

### Identity

* IAM roles
* Kubernetes RBAC
* EKS access
* Pod Identity
* Least privilege
* KMS
* Secrets

AWS currently documents EKS Pod Identity as a mechanism for associating an IAM role with a Kubernetes service account, allowing workloads to access AWS APIs without embedding credentials; AWS also recommends per-application IAM roles for isolation and least privilege. ([AWS Documentation][2])

### Operations

* Autoscaling
* Logging
* Metrics
* Alerts
* Storage
* Backups
* Upgrades
* Security

---

# 10. Phase 9 — Advanced EKS

This is where your EKS knowledge becomes **senior-level**.

### Scaling

* HPA
* VPA where appropriate
* Cluster Autoscaler
* Karpenter
* Node sizing
* Requests/limits
* Bin packing
* Pod density
* IP capacity

### Reliability

* Multi-AZ
* Pod disruption
* Node failure
* AZ failure
* Deployment safety
* Rollbacks
* Upgrade strategies
* Failure domains

### Observability

```text
                 ┌── Logs
Application ─────┼── Metrics
                 └── Traces
                       ↓
                 Observability
                       ↓
                    Alerts
                       ↓
                   Incident
```

You'll learn to investigate incidents rather than simply look at dashboards.

---

# 11. Phase 10 — Enterprise Architecture

Now we'll design complete systems.

Example:

```text
                    Users
                      │
                   Route 53
                      │
                 CloudFront
                      │
                    WAF
                      │
                     ALB
                      │
                  Ingress
                      │
                ┌──── EKS ────┐
                │             │
             Services      Workers
                │
        ┌───────┼────────┐
        ↓       ↓        ↓
       RDS     SQS       S3
```

With:

* Multiple AZs
* Account strategy
* IAM
* Security
* Observability
* CI/CD
* Terraform
* Disaster recovery
* Cost controls
* Governance

We'll also repeatedly ask:

> Why this architecture?

> Why not another architecture?

---

# 12. Phase 11 — Production Troubleshooting

This becomes a major specialization.

We'll work through incidents such as:

| Incident         | Skills exercised                               |
| ---------------- | ---------------------------------------------- |
| CrashLoopBackOff | Containers, logs, probes, application behavior |
| ImagePullBackOff | ECR, networking, IAM, image                    |
| Pending Pod      | Scheduling, resources, taints, node capacity   |
| OOMKilled        | Memory, limits, application behavior           |
| Node NotReady    | Node, kubelet, networking, infrastructure      |
| DNS failure      | CoreDNS, VPC DNS, networking                   |
| DB unreachable   | Routing, SG, DNS, application                  |
| LB not working   | Service, target registration, health checks    |
| IAM AccessDenied | IAM, trust, Pod Identity                       |
| IP exhaustion    | VPC CNI, subnet, ENI/IP capacity               |
| Scaling failure  | HPA, scheduler, nodes, Karpenter               |
| Upgrade failure  | EKS, nodes, workloads, compatibility           |

The methodology will be:

```text
Symptom
   ↓
Scope
   ↓
Evidence
   ↓
Hypotheses
   ↓
Test hypothesis
   ↓
Narrow search space
   ↓
Root cause
   ↓
Fix
   ↓
Prevention
```

That methodology is more valuable than memorizing 200 `kubectl` commands.

---

# 13. Phase 12 — Senior Architecture + Interviews

At this point interviews become **design defense**.

You'll get scenarios like:

> Design a production EKS platform for 100 microservices.

Then I'll challenge:

* Why EKS?
* Why not ECS?
* How many clusters?
* Why private nodes?
* How does ingress work?
* How do workloads get AWS permissions?
* What happens if an AZ fails?
* What happens if the VPC runs out of IPs?
* How do you upgrade?
* How do you roll back?
* How do you detect failure?
* How do you control costs?
* What is your blast radius?
* What happens if credentials leak?

The goal is to make your thinking sound like an engineer who has **operated systems**, not someone who memorized architecture diagrams.

---

# 14. AWS service priority map

This is intentionally selective.

| Service / Technology   | Priority           |             Depth | Why                                    |
| ---------------------- | ------------------ | ----------------: | -------------------------------------- |
| **VPC**                | MUST               |              Deep | Foundation of EKS networking           |
| **IAM**                | MUST               |              Deep | Security and AWS access                |
| **EC2**                | MUST               |              Deep | Understand compute/nodes               |
| **EKS**                | MUST               |         Very Deep | Your primary specialization            |
| **ECR**                | MUST               |            Medium | Container registry                     |
| **S3**                 | MUST               |            Medium | Fundamental AWS storage                |
| **EBS**                | MUST               |       Medium/Deep | Kubernetes/AWS storage                 |
| **ALB**                | MUST               |              Deep | Common EKS ingress path                |
| **Route 53**           | MUST               |            Medium | DNS                                    |
| **CloudWatch**         | MUST               |       Medium/Deep | AWS observability                      |
| **AWS CLI**            | MUST               |            Medium | Daily engineering tool                 |
| **Terraform**          | MUST               |              Deep | Infrastructure automation              |
| **Docker**             | MUST               |              Deep | Container foundation                   |
| **Kubernetes**         | MUST               |         Very Deep | EKS foundation                         |
| **Linux**              | MUST               |              Deep | Troubleshooting foundation             |
| **Git**                | MUST               |            Medium | Engineering workflow                   |
| **RDS**                | MUST               |            Medium | Common application dependency          |
| **NAT Gateway**        | MUST               | Deep conceptually | Private subnet connectivity            |
| **Security Groups**    | MUST               |              Deep | Network security                       |
| **NACLs**              | SHOULD             |            Medium | Understand, don't obsess               |
| **EFS**                | SHOULD             |            Medium | Kubernetes shared storage              |
| **Secrets Manager**    | SHOULD             |            Medium | Secrets management                     |
| **KMS**                | SHOULD             |            Medium | Encryption/key management              |
| **Systems Manager**    | SHOULD             |            Medium | EC2 operations                         |
| **CloudFront**         | SHOULD             |            Medium | Production edge architecture           |
| **WAF**                | SHOULD             |            Medium | Production security                    |
| **SQS**                | SHOULD             |            Medium | Async architecture                     |
| **SNS**                | SHOULD             |  Awareness/Medium | Messaging                              |
| **DynamoDB**           | SHOULD             |            Medium | Recognize/design basic use cases       |
| **ElastiCache**        | SHOULD             |            Medium | Common production dependency           |
| **ECS**                | SHOULD             |            Medium | Important alternative to EKS           |
| **Lambda**             | SHOULD             |  Awareness/Medium | Understand serverless alternative      |
| **EventBridge**        | AWARENESS          |             Basic | Recognize event architectures          |
| **API Gateway**        | AWARENESS          |             Basic | Recognize serverless APIs              |
| **Step Functions**     | AWARENESS          |             Basic | Recognize orchestration                |
| **DMS**                | AWARENESS          |             Basic | Migration awareness                    |
| **Transit Gateway**    | AWARENESS → SHOULD |      Basic/Medium | Enterprise networking                  |
| **Organizations**      | SHOULD             |            Medium | Enterprise accounts                    |
| **Control Tower**      | AWARENESS          |             Basic | Enterprise governance                  |
| **Direct Connect**     | AWARENESS          |             Basic | Hybrid networking                      |
| **EKS Fargate**        | AWARENESS          |      Basic/Medium | Know where it fits                     |
| **EKS Auto Mode**      | AWARENESS → later  |  TBD/deeper later | Understand current EKS operating model |
| **Karpenter**          | MUST eventually    |              Deep | Modern EKS scaling                     |
| **Cluster Autoscaler** | SHOULD             |            Medium | Understand alternative                 |
| **Prometheus**         | MUST eventually    |       Medium/Deep | Kubernetes metrics                     |
| **Grafana**            | SHOULD             |            Medium | Visualization                          |
| **OpenTelemetry**      | SHOULD             |            Medium | Modern observability                   |
| **Argo CD/GitOps**     | SHOULD             |            Medium | Introduce only after fundamentals      |

The EKS Best Practices Guide itself currently organizes guidance around security, IAM, networking, scaling, reliability, observability-related concerns, and related operational areas—exactly the dimensions we'll progressively cover rather than treating EKS as merely “create cluster + deploy YAML.” ([AWS Documentation][3])

---

# 15. Dependency map

The most important dependency chain is:

```text
Linux
 │
 ├── Processes
 ├── Filesystems
 ├── Permissions
 └── Troubleshooting
        │
        ↓
Networking Fundamentals
 │
 ├── IP
 ├── CIDR
 ├── Routing
 ├── TCP
 ├── DNS
 └── HTTP/TLS
        │
        ↓
AWS Fundamentals
 │
 ├── Accounts
 ├── Regions/AZs
 ├── IAM
 └── Resources
        │
        ↓
AWS Networking
 │
 ├── VPC
 ├── Subnets
 ├── Routes
 ├── SG
 ├── NAT
 └── ENI
        │
        ↓
AWS Compute
 │
 ├── EC2
 ├── EBS
 ├── ALB
 └── RDS
        │
        ↓
Containers
 │
 ├── Docker
 ├── Images
 ├── Registry
 └── Container networking
        │
        ↓
Kubernetes
 │
 ├── Pods
 ├── Controllers
 ├── Services
 ├── Scheduling
 └── Storage
        │
        ↓
EKS
 │
 ├── EKS control plane
 ├── Nodes
 ├── VPC CNI
 ├── IAM
 └── AWS integrations
        │
        ↓
Production EKS
 │
 ├── Security
 ├── Scaling
 ├── Observability
 ├── Storage
 ├── Reliability
 └── Upgrades
        │
        ↓
Enterprise Architecture
        │
        ↓
Senior Troubleshooting + Architecture
```

Notice something important:

**Kubernetes comes before EKS.**

And:

**AWS networking comes before serious EKS networking.**

That's deliberate.

---

# 16. Project progression

We will have one evolving application rather than nine unrelated toy projects.

| Project | Result                          |
| ------- | ------------------------------- |
| **P1**  | Linux-hosted application        |
| **P2**  | AWS EC2 application             |
| **P3**  | Multi-tier VPC architecture     |
| **P4**  | HA-ish EC2 + ALB + RDS          |
| **P5**  | Containerized application + ECR |
| **P6**  | CI/CD + Terraform               |
| **P7**  | Local Kubernetes                |
| **P8**  | First EKS application           |
| **P9**  | Production-style EKS            |
| **P10** | Enterprise EKS platform         |
| **P11** | Incident laboratory             |
| **P12** | Senior architecture capstone    |

Each project will inherit the previous one.

That gives you an actual engineering story:

> “I built it → changed it → broke it → diagnosed it → automated it → made it production-ready.”

That's much stronger than twelve disconnected tutorials.

---

# 17. Interview progression

### Level 1 — Fundamentals

> What is a VPC?

> What is a subnet?

> What is an EC2 instance?

> What is a container?

> What is a Pod?

### Level 2 — Practical

> How would you deploy an application to a private subnet?

> How does an EC2 instance reach S3?

> How do you expose a Kubernetes application?

### Level 3 — Troubleshooting

> A pod is Running but cannot reach RDS. Diagnose it.

> A deployment is stuck. What do you inspect?

### Level 4 — Architecture

> Design a highly available EKS platform.

### Level 5 — Senior

> Your EKS platform works but costs 40% more than expected. How do you investigate?

### Level 6 — Senior+ follow-up

I'll challenge assumptions:

> Why?

> What alternative did you consider?

> What's the failure mode?

> What's the blast radius?

> What happens during an AZ outage?

> What happens during an upgrade?

> How would you prove your design works?

That's where senior interviews are won.

---

# 18. What NOT to learn

This list is extremely important.

For now:

### SKIP

* Learning every AWS service
* Memorizing AWS CLI options
* Deep CloudFormation
* Deep CDK
* Multiple CI/CD platforms
* Multiple Kubernetes distributions
* Service mesh
* Istio
* Linkerd
* Advanced serverless
* Advanced DynamoDB internals
* Advanced Kafka
* Advanced database administration
* Advanced BGP
* Deep multi-region active/active architecture
* Advanced FinOps tooling
* Every CNCF project
* Every Kubernetes operator
* Every observability platform

### Specifically don't do this

```text
Terraform
   ↓
Pulumi
   ↓
CloudFormation
   ↓
CDK
   ↓
Ansible
   ↓
Crossplane
```

No.

We'll learn **Terraform deeply enough to manage infrastructure professionally**, then move on.

Likewise:

```text
Jenkins
GitHub Actions
GitLab CI
CircleCI
Argo Workflows
Tekton
...
```

No tool collection.

We'll choose a small toolset and understand the underlying CI/CD principles.

---

# 19. The engineering framework we'll use

AWS's Well-Architected Framework is useful here—not as a certification checklist, but as a way to challenge architecture decisions.

AWS currently frames architecture around six pillars:

* Operational excellence
* Security
* Reliability
* Performance efficiency
* Cost optimization
* Sustainability ([AWS Documentation][4])

We'll repeatedly use a simpler mental model:

```text
              Architecture
                   │
      ┌────────────┼────────────┐
      ↓            ↓            ↓
 Reliability    Security      Cost
      │            │            │
      └────────────┼────────────┘
                   ↓
              Operability
                   │
                   ↓
              Performance
                   │
                   ↓
             Business need
```

And AWS explicitly emphasizes making small, reversible changes and using observability/automation as part of operational excellence. ([AWS Documentation][5])

---

# 20. Cost strategy

For your labs, we'll favor:

**cheap → disposable → automated → deleted**

rather than:

**production-sized → expensive → forgotten**

We'll identify resources that can generate ongoing costs before creating them.

Particular things we'll be careful about include:

* NAT Gateways
* EKS
* Load Balancers
* EC2
* RDS
* EBS
* CloudWatch logs
* Data transfer

AWS's cost guidance emphasizes cost awareness, selecting resource size/number based on actual demand, and optimizing supply dynamically. ([AWS Documentation][6])

For every lab I'll tell you:

**CREATE → USE → VERIFY → DESTROY**

---

# 21. Definition of success

When we're finished, I want you to be able to receive this architecture:

```text
                   Internet
                      │
                  Route 53
                      │
                 CloudFront/WAF
                      │
                     ALB
                      │
                  EKS Ingress
                      │
             ┌────────┴────────┐
             │      EKS       │
             │                │
          Services         Workers
             │
      ┌──────┼──────┬──────┐
      ↓      ↓      ↓      ↓
     RDS    SQS     S3   External API
```

and independently answer:

### Architecture

* Why is each component there?
* What alternatives exist?
* Where are the failure domains?
* Where are the security boundaries?
* What's the blast radius?

### Networking

* What IP does the pod have?
* How does it reach RDS?
* How does traffic reach the pod?
* What happens when the subnet runs out of IPs?
* How does DNS work?

### IAM

* Who can access the cluster?
* Who can deploy?
* What AWS permissions does the pod have?
* Why does that pod have those permissions?

### Kubernetes

* Why is this Pod scheduled there?
* Why is it Pending?
* Why did it restart?
* Why isn't the Service routing traffic?

### Scaling

* Why didn't HPA scale?
* Why didn't new nodes appear?
* Why can't the scheduler place the Pod?
* Why is the cluster expensive?

### Operations

* How do you monitor it?
* How do you upgrade it?
* How do you roll back?
* How do you recover?

### Incidents

You can take:

> **“Production is broken.”**

and turn it into:

```text
What changed?
     ↓
What is affected?
     ↓
What evidence do we have?
     ↓
Where is the failure boundary?
     ↓
What hypothesis explains it?
     ↓
How can we prove/disprove it?
     ↓
What is the safest mitigation?
     ↓
What is the root cause?
     ↓
How do we prevent recurrence?
```

That is the final skill we're actually training.

---

# 22. Phase 0 starts now

We are **not** starting with AWS.

We're starting underneath AWS.

## Phase 0 — Linux + Networking Foundations

### Goal

By the end of Phase 0, you should be comfortable enough with Linux and networking that AWS and Kubernetes don't feel like magic.

We'll use this model:

**Simple explanation → technical explanation → build → break → troubleshoot → interview**

---

## 22.1 Why Linux matters

Imagine an EKS Pod:

```text
Pod
 │
 └── Container
       │
       └── Linux process
```

If that process:

* consumes too much memory,
* cannot open a file,
* cannot bind to a port,
* cannot resolve DNS,
* cannot connect to another service,
* crashes,
* receives a signal,

you need enough Linux knowledge to understand what's happening.

Kubernetes doesn't replace Linux knowledge.

It **automates Linux workloads**.

---

# 23. Linux: the minimum we need

Don't try to become a Linux administrator.

We need five areas.

### 1. Processes

Understand:

```text
Process
PID
Parent process
Child process
Foreground/background
Signal
Exit code
```

Important signals:

```text
SIGTERM
SIGKILL
SIGINT
SIGHUP
```

You should understand why:

```text
SIGTERM
```

and

```text
SIGKILL
```

are not equivalent.

This becomes directly relevant to Kubernetes termination and graceful shutdown.

---

### 2. Files and permissions

Understand:

```text
/
├── etc
├── var
├── tmp
├── home
├── usr
├── proc
└── dev
```

And:

```text
owner
group
permissions
```

For example:

```text
-rwxr-x---
```

You should be able to answer:

> Why can this process read the file but not write to it?

---

### 3. Processes consume resources

You need to understand:

```text
CPU
Memory
Disk
Network
```

because later Kubernetes gives you:

```yaml
resources:
  requests:
  limits:
```

Those settings make much more sense when you understand the underlying machine.

---

### 4. Services

Understand the relationship:

```text
Application process
       ↓
Listening socket
       ↓
Port
       ↓
Network interface
```

If an application listens on:

```text
127.0.0.1:8080
```

that is fundamentally different from:

```text
0.0.0.0:8080
```

That distinction will become extremely important in containers and Kubernetes.

---

### 5. Logs

Learn to distinguish:

```text
Application logs
System logs
Service manager logs
Kernel-related information
```

Because troubleshooting starts with:

> What actually happened?

not:

> Which command should I run?

---

# 24. Networking: your first major mental model

Suppose:

```text
Client
  │
  │ TCP
  ↓
Server
```

The client wants:

```text
10.0.2.15:8080
```

There are several independent questions:

### Question 1

**Can the client resolve the name?**

DNS.

### Question 2

**Can the client route packets toward the destination?**

Routing.

### Question 3

**Is the traffic allowed?**

Firewall/security rules.

### Question 4

**Is something actually listening on port 8080?**

Application/socket.

### Question 5

**Does the application respond correctly?**

Application protocol.

So:

```text
DNS
 ↓
Routing
 ↓
Firewall
 ↓
Port
 ↓
Process
 ↓
Application
```

This layered thinking will eventually become your EKS troubleshooting methodology.

---

# 25. Your first troubleshooting principle

Never say:

> “The network is not working.”

That's too vague.

Instead ask:

```text
Can I resolve the name?
        ↓
Do I have an IP?
        ↓
Can I route to it?
        ↓
Can I establish TCP?
        ↓
Is the port listening?
        ↓
Does the application respond?
```

For example:

```bash
curl http://example.com
```

isn't just a command.

You're testing multiple layers simultaneously.

Later we'll deliberately separate those layers using tools such as:

```bash
dig
nslookup
ping
ip
ip route
ss
curl
traceroute
tcpdump
```

You won't memorize them yet.

You'll learn **which question each tool answers**.

---

# 26. CIDR — your first important networking concept

Suppose you see:

```text
10.0.0.0/16
```

You need to understand what that means.

The `/16` tells you how many bits represent the network portion.

Conceptually:

```text
10.0.0.0/16
│
├── network: 10.0
└── remaining bits: host/address space
```

This becomes critical later:

```text
VPC
 ↓
Subnets
 ↓
ENIs
 ↓
Pod IPs
```

EKS networking eventually depends heavily on IP address planning. AWS's EKS networking guidance specifically warns about subnet IP availability and explains how the VPC CNI allocates pod IPs from VPC networking resources. ([AWS Documentation][1])

So CIDR is **not a networking exam topic** for us.

It's an EKS production skill.

---

# 27. Phase 0 mini-project

We're going to build a tiny Linux/networking laboratory.

## Architecture

```text
┌──────────────────────┐
│ Linux machine        │
│                      │
│  HTTP server         │
│       │              │
│       ↓              │
│    TCP :8080         │
└──────────┬───────────┘
           │
           │
      HTTP client
```

The objective isn't the application.

The objective is answering:

1. What process is running?
2. What PID does it have?
3. What port is it listening on?
4. Which address is it bound to?
5. Can another machine reach it?
6. What happens if we stop the process?
7. What happens if we change the listening address?
8. What happens if the port is blocked?
9. How do we prove where the failure occurs?

---

# 28. Phase 0 first exercise

Before we go any further, I want you to **think like the engineer**, not run commands blindly.

Imagine this:

```text
Application
   ↓
Listening on port 8080
   ↓
Client tries to connect
   ↓
Connection fails
```

There are at least several possible causes.

Your task:

### Challenge 1

Give me **5 possible reasons** why the client cannot connect to the application.

Don't search for the answer.

Think in layers.

For each one, tell me:

```text
Possible cause:
How I would verify it:
```

---

### Challenge 2

Explain this in your own words:

```text
127.0.0.1:8080
```

versus:

```text
0.0.0.0:8080
```

Why might an application work locally but be unreachable from another machine?

---

### Challenge 3 — interview

You're in an interview.

The interviewer says:

> **“An application is running on an EC2 instance, but users cannot access it. What do you check?”**

Don't give me 30 commands.

Give me your **investigation order**.

---

## Your first milestone

Don't worry about AWS yet.

If you can reason correctly through these three questions, we'll move into hands-on Linux/networking work and progressively introduce AWS.

**Do not move ahead by reading. Demonstrate the reasoning first.**

[1]: https://docs.aws.amazon.com/eks/latest/best-practices/networking.html?utm_source=chatgpt.com "Best Practices for Networking - Amazon EKS"
[2]: https://docs.aws.amazon.com/eks/latest/userguide/pod-identities.html?utm_source=chatgpt.com "Learn how EKS Pod Identity grants pods access to AWS services - Amazon EKS"
[3]: https://docs.aws.amazon.com/eks/latest/best-practices/introduction.html?utm_source=chatgpt.com "Amazon EKS Best Practices Guide - Amazon EKS"
[4]: https://docs.aws.amazon.com/wellarchitected/2024-06-27/framework/definitions.html?utm_source=chatgpt.com "Definitions - AWS Well-Architected Framework"
[5]: https://docs.aws.amazon.com/wellarchitected/latest/operational-excellence-pillar/operational-excellence.html?utm_source=chatgpt.com "Operational excellence - Operational Excellence Pillar"
[6]: https://docs.aws.amazon.com/wellarchitected/2023-10-03/framework/cost-optimization.html?utm_source=chatgpt.com "Cost optimization - AWS Well-Architected Framework"
