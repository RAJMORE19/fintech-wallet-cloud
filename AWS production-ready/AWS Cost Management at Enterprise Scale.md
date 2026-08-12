AWS Cost Management at Enterprise Scale

For 100+ microservices, AWS cost management should be treated as an engineering + finance operating model, not simply an AWS billing exercise.

The core principle is:

Every meaningful AWS dollar should be attributable to an owner, environment, workload, and business/service unit—and teams should have mechanisms to control that spend.

1. Track and attribute costs to microservices and teams

Use three layers of attribution:

A. AWS accounts — strongest ownership boundary

A practical structure is:

AWS Organization
├── OU: Production
│   ├── prod-payments
│   ├── prod-orders
│   └── prod-platform
├── OU: NonProduction
│   ├── dev
│   ├── staging
│   └── qa
├── OU: Security
├── OU: SharedServices
└── OU: Sandbox


Don't create an AWS account for every microservice. Instead, use accounts where they provide a meaningful security, operational, or ownership boundary.

Use AWS Organizations + OUs + consolidated billing.

B. Tags — workload-level attribution

Establish mandatory tags such as:

Application       = payments
Service           = payment-api
Team              = payments-team
Environment       = production
CostCenter        = CC-1042
Owner             = payments-platform
BusinessUnit      = commerce
ManagedBy         = terraform


Use AWS Tag Policies and AWS Organizations to standardize tagging, and enforce mandatory tags through IaC/policies where possible.

Important: don't rely exclusively on tags. Some AWS costs don't map cleanly to individual resources, particularly networking and shared services.

C. AWS Cost Categories

Use AWS Cost Categories to create business-friendly groupings such as:

Payments
Orders
Customer Platform
Data Platform
Shared Infrastructure
Security
Observability


Then use AWS Cost Explorer, Cost & Usage Reports (CUR) / AWS Data Exports, and dashboards to analyze spending.

For enterprise FinOps, CUR data is particularly important because it gives you detailed billing data that can be aggregated in an analytics platform.

2. Compute cost control

For EC2/ECS/EKS/Lambda:

First: eliminate waste

Look for:

Idle EC2 instances
Low CPU/memory utilization
Oversized instances
Old instance families
Unused EBS volumes
Unattached Elastic IPs
Idle load balancers
Kubernetes requests/limits significantly above actual usage
Lambda functions with inefficient memory/runtime configuration

Use:

AWS Compute Optimizer
AWS Cost Explorer
AWS Trusted Advisor
CloudWatch metrics
EKS/Kubernetes metrics for container workloads
Then automate scaling

Use:

EC2 Auto Scaling
ECS Service Auto Scaling
EKS Cluster Autoscaler/Karpenter
Lambda concurrency controls
Scheduled scaling for predictable workloads

Don't run production capacity for peak traffic 24/7 if the workload is highly variable.

3. Savings Plans, Reserved Instances and Spot

Use commitment discounts after understanding your baseline.

Savings Plans

For relatively stable compute consumption, evaluate:

Compute Savings Plans
EC2 Instance Savings Plans

They are generally more flexible than traditional Reserved Instances.

Reserved Instances

Useful where workload characteristics are stable, especially certain database workloads.

Examples:

RDS
ElastiCache
OpenSearch
other eligible services
Spot

Use Spot for workloads that tolerate interruption:

Batch processing
CI/CD workers
Data processing
Stateless asynchronous workloads
Fault-tolerant Kubernetes/ECS workloads

A common enterprise strategy is:

Baseline capacity → Savings Plan / RI
Normal variable capacity → On-demand
Interruptible workloads → Spot


Don't purchase commitments simply because utilization is currently high. First establish whether that utilization is structural or waste.

4. Database cost management

Databases can become one of the largest costs.

For RDS/Aurora:

Right-size instance classes
Review CPU, memory, connections and I/O
Delete unused databases
Stop non-production databases where appropriate
Use Aurora Serverless where workload characteristics justify it
Review storage growth
Review backup retention
Evaluate Reserved Instances for stable workloads
Avoid unnecessary Multi-AZ deployments for workloads that don't require them

For DynamoDB:

Review provisioned capacity
Use on-demand for unpredictable workloads
Use provisioned + auto scaling for predictable workloads
Examine hot partitions
Review unnecessary indexes
Monitor storage growth

For OpenSearch:

Right-size instances
Control shard counts
Configure retention
Move older data to cheaper storage where appropriate
Don't retain unlimited application logs/search data.
5. Storage costs

Storage is often a silent source of waste.

Review:

S3
Lifecycle policies
Intelligent-Tiering
Glacier tiers for archival data
Incomplete multipart uploads
Old versions
Unnecessary replication
Excessive retention
EBS

Find:

Unattached volumes
Oversized volumes
Old snapshots
Excessive snapshot retention

Use AWS Data Lifecycle Manager where appropriate.

A good rule:

Every production data-retention policy should have an explicit business reason.

6. Networking is frequently underestimated

At 100+ microservices, networking can become surprisingly expensive.

Pay attention to:

NAT Gateway
Cross-AZ traffic
Cross-region traffic
Internet egress
Inter-region replication
Load balancers
Transit Gateway
PrivateLink
VPC endpoints

A classic example is:

EKS → NAT Gateway → AWS service


If large amounts of traffic are unnecessarily traversing NAT Gateway, costs can become substantial.

Use appropriate VPC endpoints for AWS services where economically and architecturally justified.

Also examine whether services communicate across Availability Zones unnecessarily.

7. Logs and observability

This is a major cost trap in microservice architectures.

With 100+ services, don't blindly log everything at maximum verbosity.

Control:

CloudWatch Logs ingestion
Log retention
Metric cardinality
Custom metrics
X-Ray tracing
OpenSearch ingestion/storage
Third-party observability platforms

Example:

Production application logs → 30 days
Security/audit logs          → longer retention
Debug logs                   → short retention
High-volume access logs      → sampling/aggregation


Use CloudWatch Logs retention policies, filtering, sampling and appropriate archival strategies.

Teams should know:

"Our observability stack costs $X/month."

It should not be treated as an invisible platform cost.

8. Finding unused and over-provisioned resources

Create a recurring waste report.

Typical candidates:

EC2 with consistently low utilization
RDS with oversized instance class
EBS unattached volumes
Old snapshots
Idle load balancers
Unused Elastic IPs
Unused NAT gateways
Idle EKS node capacity
Underutilized OpenSearch domains
Unused development environments
Excessive log retention
Unused public IP resources


Combine:

AWS Cost Explorer
AWS Compute Optimizer
AWS Trusted Advisor
CloudWatch
AWS Resource Explorer
AWS Config
AWS Cost Optimization Hub

Don't automatically delete everything flagged as "unused." Establish an owner and validation workflow.

9. Budgets, alerts and anomaly detection

Set budgets at several levels.

Organization
AWS total budget
Production budget
Non-production budget

Team/service
Payments
Orders
Search
Data Platform
Observability

Environment
Production
Staging
Development


Use AWS Budgets for threshold-based alerts.

For unexpected spending patterns, use AWS Cost Anomaly Detection.

Example:

Expected Payments spend: $40K/month

Anomaly:
Payments spend suddenly increases by 45%

→ AWS anomaly detection
→ SNS/email/Slack
→ FinOps + service owner
→ investigate deployment/resource change


Don't wait until the monthly invoice arrives.

10. Who owns cost?

A mature model is FinOps is centralized, cost ownership is distributed.

FinOps team

Responsible for:

Governance
Cost reporting
Savings opportunities
Forecasting
Commitment strategy
Cost allocation
Standards
Executive reporting
Platform/Cloud team

Responsible for:

Infrastructure efficiency
Scaling
Architecture patterns
Shared infrastructure
Networking
Observability platform efficiency
Application teams

Responsible for:

Their services' resource consumption
Right-sizing
Application efficiency
Log volume
Database usage
Scaling configuration
Engineering leadership

Responsible for:

Cost targets
Architecture decisions
Trade-offs between performance/reliability/cost

The important concept is showback/chargeback.

Teams should see:

"Your services consumed $18,400 this month."

rather than:

"Engineering AWS bill = $500K."

11. Estimating the cost of a new microservice

Before deploying a service, create a cost estimate as part of the architecture review.

For example:

Service: payment-api

ECS/EC2 compute       $1,800
ALB                    $150
RDS/Aurora             $900
S3                      $80
CloudWatch              $180
Data transfer           $250
NAT/VPC                 $200
Other                   $100
--------------------------------
Estimated/month       ~$3,660


Then calculate:

Baseline
+ normal growth
+ peak capacity
+ data growth
+ observability
+ networking
+ backup/DR


Use the AWS Pricing Calculator before deployment.

But don't stop at infrastructure pricing.

Ask:

How many requests/sec?
Payload size?
Data stored?
Database IOPS?
Number of AZs?
Expected log GB/day?
Retention?
Network egress?
Peak traffic?
Scaling range?

The workload characteristics drive the real cost.

12. Practical architecture for 100+ microservices

A scalable operating model looks like this:

                    AWS Organizations
                           │
                    OUs / AWS Accounts
                           │
                ┌──────────┴──────────┐
                │                     │
          Production              Non-Production
                │                     │
        ┌───────┴───────┐       Dev / QA / Staging
        │               │
   Service resources   Shared services
        │
        ▼
Mandatory Cost Tags
        │
        ▼
AWS Billing / CUR / Data Exports
        │
        ▼
Cost Categories
        │
        ▼
FinOps Dashboard
        │
   ┌────┴──────────────┐
   ▼                   ▼
Teams              Leadership
   │
   ▼
Budgets + Alerts + Optimization


A good enterprise platform also creates golden paths.

For example, instead of letting every team design its own infrastructure:

Company-standard ECS service
Company-standard EKS service
Company-standard RDS pattern
Company-standard logging
Company-standard monitoring
Company-standard tagging


The platform team builds these patterns with sensible cost defaults.

That prevents 100 teams from independently making 100 expensive architectural decisions.

13. How a company could reduce AWS cost by 20–30%

Suppose current spend is:

$1,000,000/month

A realistic optimization program might find:

Area	Monthly saving
EC2/EKS rightsizing	$90K
Savings Plans	$80K
Non-prod scheduling	$45K
RDS optimization	$40K
S3/storage lifecycle	$25K
NAT/network optimization	$30K
Logs/observability	$35K
Removing unused resources	$20K
Total	$365K

New spend:

~$635K/month

That's a 36.5% theoretical opportunity, but after implementation constraints and safety margins, a 20–30% sustainable reduction is quite realistic for an environment with significant existing waste.

The important part is that this isn't achieved through one trick.

It's usually:

rightsizing + scaling + commitments + waste removal + storage optimization + networking + observability + governance.

Production AWS Cost Management Checklist
 Every production workload has an accountable owner/team.
 Mandatory Application, Service, Team, Environment, and CostCenter tags are standardized.
 AWS Organizations/OUs/accounts provide clear ownership boundaries.
 Cost Categories map AWS spending to teams/business units.
 AWS Cost Explorer and CUR/Data Exports are available to FinOps.
 AWS Budgets exist for organization, environments, and major teams.
 AWS Cost Anomaly Detection is enabled.
 Compute rightsizing is reviewed regularly.
 Auto Scaling is configured appropriately.
 Savings Plans/RIs are reviewed against stable baseline usage.
 Spot is used for suitable fault-tolerant workloads.
 Unattached/idle resources are regularly identified and remediated.
 RDS/Aurora capacity, storage, backups and retention are reviewed.
 S3 lifecycle and storage-class policies are implemented.
 NAT Gateway, cross-AZ and internet-egress costs are monitored.
 CloudWatch/OpenSearch/log retention is controlled.
 Non-production environments are scheduled down when appropriate.
 Every new major service has a pre-deployment cost estimate.
 Teams receive monthly showback for their AWS consumption.
 FinOps runs a recurring optimization review with engineering.
 Cost is treated as a production engineering metric alongside availability, latency and security.
