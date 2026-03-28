# Operations

**Purpose**: Plan deployment, configure monitoring/observability, establish production readiness, and **generate implementation-ready code** for all operational concerns.

## Prerequisites
- Build and Test stage must be complete for all units
- All tests must pass (unit, integration, performance as applicable)
- Build and Test summary must indicate readiness for operations

---

## Step 1: Analyze Deployment Context and Collect Prior Decisions

- [ ] Read build-and-test summary from `aidlc-docs/construction/build-and-test/build-and-test-summary.md`
- [ ] **GATE CHECK**: Verify `Ready for Operations` is `Yes` in build-and-test-summary.md. If `No`: present blocking message explaining which tests failed and do NOT proceed until resolved.
- [ ] Read integration state from `aidlc-docs/construction/build-and-test/integration-state.md` for per-unit details (TDD selection, stage progression, build/test results)
- [ ] Read infrastructure design artifacts (if exists) from `aidlc-docs/construction/{unit-name}/infrastructure-design/`
- [ ] Read NFR requirements (if exists) for availability, scalability, security constraints
- [ ] Identify all deployable units and their dependencies
- [ ] Determine deployment target (cloud provider, on-premise, hybrid)

### Context Auto-Collection from Prior Phases

**DIRECTIVE**: Scan all prior phase artifacts and extract already-decided values. These MUST NOT be asked again as questions — they are treated as confirmed inputs.

- [ ] **From NFR Requirements** — collect:
  - Performance thresholds (response time targets, concurrent users, throughput)
  - Availability targets (SLA %, RTO, RPO)
  - Security requirements (encryption, auth method)
  - Monitoring tool selections (CloudWatch, X-Ray, Datadog, etc.)
- [ ] **From Infrastructure Design / Deployment Architecture** — collect:
  - AWS service list with specifications
  - Auto Scaling policies (already-defined thresholds)
  - Network configuration (VPC, Subnets)
  - Deployment strategy (if already defined)
  - CI/CD tool selections (GitHub Actions, CodeDeploy, etc.)
- [ ] **From IaC code (Terraform/CDK/CloudFormation)** — collect:
  - Resource names and identifiers (e.g., `aws_autoscaling_group.main`)
  - Existing health check configurations
  - Existing security group rules
  - Variable naming conventions and patterns
- [ ] **From Tech Stack Decisions** — collect:
  - CI/CD tooling
  - Logging framework
  - Runtime environment

- [ ] Save collected context to `aidlc-docs/operations/plans/context-inventory.md`:

```markdown
# Operations Context Inventory

## Confirmed Values from Prior Phases
| Item | Value | Source |
|------|-------|--------|
| [metric/config name] | [confirmed value] | [source file] |

## Undecided Items (Questions Required)
| Item | Reason Needed |
|------|--------------|
| [item] | [why this needs user input] |
```

---

## Step 2: Determine Operations Depth

Assess project complexity to determine appropriate detail level for operations artifacts. Follow `common/depth-levels.md` principles: "Create exactly the detail needed for the problem at hand - no more, no less."

**Factors to consider**:
- Number of deployable units
- Infrastructure complexity (single service vs distributed system)
- Production vs non-production deployment target
- Regulatory or compliance requirements
- Team operational maturity

**Depth determines detail level within artifacts, not which sub-stages execute**:
- Deployment Planning: ALWAYS
- Monitoring & Observability Implementation: CONDITIONAL - if production deployment or user requests it
- CI/CD Pipeline Code Generation: CONDITIONAL - if CI/CD tool is identified in prior phases or user requests it
- Production Readiness Checklist: CONDITIONAL - if production deployment or user requests it
- Runbook & Automation Scripts: CONDITIONAL - if complex system with multiple services or user requests it

- [ ] Document depth assessment rationale
- [ ] Save as `aidlc-docs/operations/plans/operations-plan.md`

---

## Step 3: Generate Context-Appropriate Questions

**DIRECTIVE**: Analyze the build artifacts, infrastructure design, and context inventory to generate questions relevant to THIS specific project's deployment needs. Follow `common/question-format-guide.md` format for all questions. Follow `common/overconfidence-prevention.md` principles — evaluate ALL relevant categories and default to asking when in doubt.

**CRITICAL**: Do NOT ask questions about items already confirmed in the context inventory. Only ask about undecided items.

- [ ] Generate questions using [Answer]: tag format with multiple choice options (A, B, C, D, E)
- [ ] Focus on ambiguities and missing information specific to deployment
- [ ] Evaluate all categories below — generate questions for each category where user input is needed

### Strategy-Level Questions (generate ONLY if not already decided in prior phases)

**Question categories to evaluate** (skip if already confirmed in context inventory):

#### Deployment Strategy (skip if deployment-architecture.md already defines this)
```markdown
## Question: Deployment Strategy
What deployment strategy should be used?

A) Blue/green deployment — zero-downtime with instant rollback
B) Canary deployment — gradual traffic shift with monitoring
C) Rolling update — incremental replacement of instances
D) Direct replacement — simple swap (acceptable downtime)
X) Other (please describe)

[Answer]:
```

#### Environment Configuration (skip if already defined)
```markdown
## Question: Environment Setup
How many environments are needed?

A) Single environment (production only)
B) Two environments (staging + production)
C) Three environments (dev + staging + production)
D) Custom environment setup
X) Other (please describe)

[Answer]:
```

### Implementation-Level Questions (ALWAYS evaluate — these are typically NOT decided in prior phases)

#### Alert Notification Channel
```markdown
## Question: Alert Notification Target
Where should alarms and alerts be sent?

A) SNS → Email (team email address required)
B) SNS → Slack Webhook (Webhook URL required)
C) SNS → PagerDuty Integration (Integration Key required)
D) SNS → Email + Slack (both)
E) No alerting needed at this stage
X) Other (please describe)

[Answer]:
```

#### Log Retention and Structure
```markdown
## Question: Log Retention and Structure
How should application logs be configured?

A) 30-day retention, basic log groups
B) 90-day retention, structured JSON logging with Metric Filters
C) 365-day retention, with S3 archival
D) CloudWatch + OpenSearch (log analysis dashboards included)
X) Other (please describe)

[Answer]:
```

#### CI/CD Pipeline Scope
```markdown
## Question: CI/CD Pipeline Configuration
What stages should the CI/CD pipeline include?

A) Basic: lint → test → deploy
B) Standard: lint → test → build → staging deploy → smoke test → prod deploy
C) Advanced: lint → SAST → test → build → staging → integration test → canary → prod
D) Use pipeline structure already defined in deployment-architecture.md
X) Other (please describe)

[Answer]:
```

#### Dashboard Scope
```markdown
## Question: Monitoring Dashboard
What level of dashboard should be generated?

A) Single overview dashboard (key metrics only)
B) Per-service dashboards (API, DB, Cache separately)
C) Overview + per-service + business metrics dashboards
D) No dashboard needed (use CloudWatch console directly)
X) Other (please describe)

[Answer]:
```

#### Rollback Strategy (skip if already defined)
```markdown
## Question: Rollback Approach
What rollback strategy should be in place?

A) Automated rollback on health check failure
B) Manual rollback with documented procedure
C) Both automated triggers and manual procedure
D) No rollback needed (non-critical deployment)
X) Other (please describe)

[Answer]:
```

**Note**: These are example questions. Generate ONLY the questions relevant to this specific project context. Add additional questions for security, compliance, DNS, secrets management, etc. as the project requires. Remove questions where the answer is already confirmed in the context inventory.

- [ ] Save questions to `aidlc-docs/operations/plans/operations-questions.md`

---

## Step 4: Collect and Analyze Answers

- [ ] In autonomous mode: AI self-answers all [Answer]: tags based on context analysis
- [ ] Review for vague or ambiguous responses (look for: "depends", "maybe", "not sure", "mix of", "somewhere between")
- [ ] Create follow-up questions for ANY unclear responses per `common/overconfidence-prevention.md`
- [ ] Validate answers against infrastructure design (if exists)
- [ ] Do NOT proceed until all ambiguities are resolved

---

## Step 5: Generate Implementation Plan (Review Gate)

**DIRECTIVE**: Before generating any code, present a complete implementation plan for user review and approval. This mirrors the Construction phase's plan-then-execute pattern.

Create `aidlc-docs/operations/plans/implementation-plan.md`:

```markdown
# Operations Implementation Plan

## Files to Generate
| File | Purpose | Impact on Existing Code |
|------|---------|------------------------|
| [file path] | [purpose] | [new file / modifies existing] |

## IaC Resources to Create
| Resource | Type | Configuration Summary |
|----------|------|----------------------|
| [resource name] | [aws_cloudwatch_metric_alarm, etc.] | [brief config] |

## Estimated Cost Impact
| Item | Estimated Monthly Cost |
|------|----------------------|
| [CloudWatch Alarms] | [$ amount] |
| [CloudWatch Dashboard] | [$ amount] |
| [CloudWatch Logs retention] | [$ amount] |
| **Total** | **[$ amount]** |

## Integration Notes
- How new files integrate with existing IaC code
- Variable reuse and naming convention alignment
- Dependency on existing resources
```

- [ ] **AUTONOMOUS MODE**: Self-review implementation plan for completeness, auto-approve and proceed to code generation immediately.

---

## Step 6: Generate Deployment Plan

Create `aidlc-docs/operations/deployment-plan.md`:

**DIRECTIVE**: Use confirmed values from context inventory. Only use placeholders for items that genuinely have no confirmed value.

```markdown
# Deployment Plan

## Deployment Overview
- **Strategy**: [Use value from context inventory or user answer]
- **Target Environment**: [From infrastructure design]
- **Deployable Units**: [From build-and-test summary]
- **Deployment Order**: [Sequence considering dependencies from application design]

## Prerequisites
- **Infrastructure**: [From infrastructure design — specific resource names]
- **Credentials**: [Required access and secrets configured]
- **Dependencies**: [External services available]
- **Approvals**: [Required sign-offs]

## Environment Configuration

### Environment Variables
| Variable | Description | Dev | Staging | Prod |
|----------|-------------|-----|---------|------|
| [VAR_NAME] | [Description] | [Value] | [Value] | [Value] |

> **⚠️ SECURITY**: Do NOT include actual secret values (passwords, API keys, tokens) in this table. Reference secrets by name only and manage them through the Secrets Management approach below.

### Secrets Management
- **Approach**: [AWS Secrets Manager, Parameter Store, Vault, etc.]
- **Secrets List**: [List secrets without values]

## Deployment Steps

### 1. Pre-Deployment Checks
[Commands/steps to verify readiness]

### 2. Deploy Infrastructure (if IaC)
[Commands to provision/update infrastructure]

### 3. Deploy Application Units
[Commands per unit in dependency order]

### 4. Post-Deployment Verification
[Smoke tests, health checks, validation steps]

### 5. DNS/Traffic Cutover (if applicable)
[Steps to route traffic to new deployment]

## Rollback Procedure

### Automated Rollback Triggers
- [Trigger 1: e.g., error rate > threshold from NFR]
- [Trigger 2: e.g., health check failures]

### Manual Rollback Steps
1. [Step-by-step rollback procedure]
2. [Data rollback considerations]
3. [Verification after rollback]
```

---

## Step 7: Generate Monitoring & Observability Implementation (If Applicable)

**DIRECTIVE**: Generate BOTH documentation AND implementation-ready IaC code. Use confirmed values from context inventory for all thresholds and configurations.

### Step 7a: Generate Monitoring Documentation

Create `aidlc-docs/operations/monitoring-setup.md` with confirmed values (not placeholders):

```markdown
# Monitoring & Observability Setup

## Metrics

### Application Metrics
| Metric | Description | Threshold | Alert Target |
|--------|-------------|-----------|--------------|
| Response Time (p95/p99) | [From NFR] | < [confirmed value]ms | [From user answer] |
| Error Rate | [Description] | < [confirmed value]% | [From user answer] |
| Throughput | [Description] | > [confirmed value] rps | [From user answer] |

### Infrastructure Metrics
| Metric | Description | Threshold | Alert Target |
|--------|-------------|-----------|--------------|
| CPU Utilization | [Description] | < [from ASG policy]% | [From user answer] |
| Memory Usage | [Description] | < [value]% | [From user answer] |
| DB Connections | [Description] | < [value] | [From user answer] |

## Logging
- **Log Level**: [From tech stack / user answer]
- **Log Format**: [Structured JSON recommended]
- **Log Aggregation**: [From NFR / tech stack]
- **Log Retention**: [From user answer]

## Tracing
- **Tracing Tool**: [From NFR — e.g., X-Ray]
- **Sampling Rate**: [value]%

## Dashboards
- [Dashboard scope from user answer]
```

### Step 7b: Generate Monitoring IaC Code

**DIRECTIVE**: Generate actual IaC code files that can be applied with `terraform apply`, `cdk deploy`, or equivalent. Match the existing IaC tool, style, naming conventions, and variable patterns found in the project.

- [ ] Identify existing IaC tool and patterns from construction artifacts
- [ ] Generate monitoring IaC code in the **same directory and style** as existing IaC:

**For Terraform projects**, generate files alongside existing `.tf` files:

- [ ] `monitoring.tf` — CloudWatch Metric Alarms:
  - Alarms for each critical metric identified in monitoring-setup.md
  - Reference existing resources by their Terraform identifiers (e.g., `aws_autoscaling_group.main.name`)
  - Use thresholds from context inventory (confirmed NFR values)

- [ ] `sns.tf` (or append to existing) — SNS Topic + Subscriptions:
  - Alarm notification topic
  - Subscription type based on user answer (email, Slack webhook, PagerDuty)

- [ ] `dashboard.tf` — CloudWatch Dashboard (if user requested):
  - Dashboard JSON definition with widgets for selected metrics
  - Reference existing resource identifiers

- [ ] `log_groups.tf` — CloudWatch Log Groups:
  - Application log group with retention from user answer
  - Metric Filters for error pattern detection (if user selected structured logging)

- [ ] Update `variables.tf` — Add new variables needed by monitoring resources (e.g., `alert_email`, `slack_webhook_url`)

**For CDK projects**, generate constructs in the existing CDK structure.
**For CloudFormation**, generate additional template resources.

### IaC Code Generation Principles
- **Match existing style**: Use same naming conventions, variable patterns, and formatting as existing IaC code
- **Reference, don't duplicate**: Reference existing resources by their identifiers — never recreate them
- **Reuse existing variables**: Use `var.project_name`, `var.environment`, etc. from existing code
- **New variables only when needed**: Add to existing `variables.tf` rather than creating separate variable files
- **No secrets in code**: Reference secrets by name only (e.g., `var.alert_email`), never embed values

---

## Step 8: Generate CI/CD Pipeline Code (If Applicable)

**Execute IF**: CI/CD tool is identified in prior phases (tech stack decisions, deployment architecture) or user requests it.

**DIRECTIVE**: Generate actual pipeline configuration files that can be committed and executed. Use the CI/CD tool already selected in prior phases.

### Step 8a: Generate Pipeline Documentation

Create `aidlc-docs/operations/cicd-pipeline.md`:

```markdown
# CI/CD Pipeline

## Pipeline Overview
- **Tool**: [From tech stack — e.g., GitHub Actions, GitLab CI, CodePipeline]
- **Trigger**: [e.g., push to main branch]
- **Stages**: [From user answer — basic/standard/advanced]

## Pipeline Stages
| Stage | Description | Failure Action |
|-------|-------------|----------------|
| [stage name] | [what it does] | [block/warn/notify] |

## Secrets Required
| Secret Name | Purpose | Where to Configure |
|-------------|---------|-------------------|
| [secret] | [purpose] | [GitHub Secrets / AWS Secrets Manager / etc.] |
```

### Step 8b: Generate Pipeline Code

- [ ] Generate pipeline configuration file in the project root:
  - **GitHub Actions**: `.github/workflows/deploy.yml`
  - **GitLab CI**: `.gitlab-ci.yml`
  - **CodePipeline**: `pipeline.tf` or `buildspec.yml`

- [ ] Generate or validate deployment scripts:
  - Validate existing `appspec.yml` (if CodeDeploy) — add health check and rollback hooks if missing
  - Generate deployment hook scripts (`scripts/deploy/`) as needed

### Pipeline Code Generation Principles
- **Reflect prior decisions**: Implement the pipeline structure defined in deployment-architecture.md
- **Secrets by reference**: Use `${{ secrets.NAME }}` (GitHub) or equivalent — never embed values
- **Environment branching**: Include dev/staging/prod separation if multiple environments were selected
- **Match existing patterns**: If `appspec.yml` or deploy scripts already exist, extend rather than replace

---

## Step 9: Generate Production Readiness Checklist (If Applicable)

Create `aidlc-docs/operations/production-readiness-checklist.md`:

```markdown
# Production Readiness Checklist

## Build & Test
- [ ] All unit tests pass
- [ ] All integration tests pass
- [ ] Performance tests meet NFR thresholds
- [ ] Security scan completed with no critical findings

## Deployment
- [ ] Deployment plan reviewed and approved
- [ ] Rollback procedure documented and tested
- [ ] Environment variables and secrets configured
- [ ] Infrastructure provisioned and validated

## Monitoring & Observability
- [ ] Monitoring IaC code reviewed and applied
- [ ] Alerting rules configured and tested
- [ ] Log aggregation configured
- [ ] Dashboards created (if applicable)

## CI/CD
- [ ] Pipeline configuration committed and tested
- [ ] Pipeline runs successfully on test branch
- [ ] Deployment scripts validated

## Security
- [ ] SSL/TLS certificates configured
- [ ] Network security rules applied
- [ ] Authentication/authorization verified
- [ ] Dependency vulnerabilities addressed

## Documentation
- [ ] API documentation up to date
- [ ] Runbook available (if applicable)
- [ ] Architecture diagram current
- [ ] On-call procedures documented

## Sign-off
- [ ] Technical lead approval
- [ ] Product owner approval
- [ ] Security review (if required)
- [ ] Ready for production deployment
```

---

## Step 10: Generate Runbook & Automation Scripts (If Applicable — Complex Systems)

**DIRECTIVE**: Generate BOTH a runbook document AND executable automation scripts. Each runbook procedure should reference a corresponding script where applicable.

### Step 10a: Generate Runbook Document

Create `aidlc-docs/operations/runbook.md`:

```markdown
# Operations Runbook

## Service Overview
- **Service Name**: [Name]
- **Owner**: [Team/Person]
- **Dependencies**: [List external dependencies]
- **SLA**: [From NFR — confirmed availability target]

## Common Operational Procedures

### Scaling Up/Down
- **Script**: `scripts/ops/scale.sh`
1. [Steps to scale]

### Restarting Services
1. [Steps to restart safely]

### Database Maintenance
- **Script**: `scripts/ops/db-backup.sh`
1. [Backup, migration, cleanup procedures]

## Incident Response

### High Error Rate
- **Detection**: [Alarm name from monitoring.tf]
- **Script**: `scripts/ops/health-check.sh`
- **Diagnosis**: [Steps to diagnose]
- **Resolution**: [Steps to resolve]
- **Escalation**: [When and who to escalate to]

### Performance Degradation
- **Detection**: [Alarm name from monitoring.tf]
- **Diagnosis**: [Steps to diagnose]
- **Resolution**: [Steps to resolve]

### Service Outage
- **Detection**: [Alarm name from monitoring.tf]
- **Immediate Actions**: [First response steps]
- **Recovery**: `scripts/ops/rollback.sh`
- **Post-Incident**: [Review and prevention]

## On-Call & Escalation

### Escalation Matrix
| Severity | Response Time | Escalation Path |
|----------|--------------|-----------------|
| SEV-1 (Outage) | 15 min | On-call → Team Lead → Engineering Manager |
| SEV-2 (Degraded) | 30 min | On-call → Team Lead |
| SEV-3 (Minor) | 4 hours | On-call |

## Disaster Recovery
- **RPO**: [From NFR — confirmed value]
- **RTO**: [From NFR — confirmed value]

### DR Procedure
1. [Activate DR plan and notify stakeholders]
2. [Failover to secondary region/environment]
3. [Verify data integrity and service health]
4. [Communicate status to stakeholders]
5. [Post-recovery: failback and review]
```

### Step 10b: Generate Automation Scripts

**DIRECTIVE**: Generate executable scripts that implement the runbook procedures. Scripts should use the cloud CLI tools and service identifiers from the existing IaC code.

- [ ] `scripts/ops/health-check.sh` — Service health verification
  - Check application health endpoint
  - Check database connectivity
  - Check cache connectivity
  - Report status summary

- [ ] `scripts/ops/rollback.sh` — Rollback automation
  - Identify previous deployment version
  - Execute rollback via deployment tool (CodeDeploy, etc.)
  - Verify health after rollback

- [ ] `scripts/ops/scale.sh` — Manual scaling
  - Accept desired capacity as parameter
  - Update ASG desired count
  - Wait for instances to be healthy

- [ ] `scripts/ops/db-backup.sh` — Database backup
  - Create manual snapshot with timestamp
  - Verify snapshot completion

- [ ] `scripts/ops/log-search.sh` — Log search utility
  - Accept time range and search pattern
  - Query CloudWatch Logs Insights
  - Output formatted results

### Script Generation Principles
- **Use resource identifiers from IaC**: Reference the same names/IDs defined in Terraform/CDK
- **Parameterize**: Accept environment (dev/staging/prod) as parameter
- **Error handling**: Include basic error checking and exit codes
- **Idempotent**: Scripts should be safe to run multiple times

---

## Step 11: Verify Extension Rule Compliance

**MANDATORY**: Before presenting completion, verify compliance with all enabled extension rules.

- [ ] Load enabled extensions from `aidlc-docs/aidlc-state.md` under `## Extension Configuration`
- [ ] For each enabled extension, evaluate applicability to operations artifacts
- [ ] Mark applicable rules as compliant/non-compliant
- [ ] Mark non-applicable rules as N/A with brief rationale
- [ ] If any applicable rule is non-compliant: resolve before proceeding (blocking finding)
- [ ] Include compliance summary in operations-summary.md

---

## Step 12: Generate Operations Summary

Create `aidlc-docs/operations/operations-summary.md`:

```markdown
# Operations Summary

## Deployment
- **Strategy**: [Deployment strategy]
- **Target**: [Deployment target]
- **Units**: [Number of deployable units]
- **Status**: [Ready/Not Ready]

## Monitoring Implementation
- **Alarms**: [Number of alarms created / N/A]
- **Dashboard**: [Created/Not Created/N/A]
- **Log Groups**: [Configured/Not Configured/N/A]
- **IaC Files Generated**: [List of .tf/.ts files]

## CI/CD Pipeline
- **Pipeline**: [Generated/Not Generated/N/A]
- **Tool**: [GitHub Actions/GitLab CI/CodePipeline/etc.]
- **Stages**: [List of pipeline stages]

## Automation Scripts
- **Scripts Generated**: [List of scripts / N/A]

## Production Readiness
- **Checklist Status**: [X/Y items complete / N/A]
- **Blockers**: [List any blockers]
- **Sign-offs**: [Pending/Complete/N/A]

## Extension Rule Compliance
| Rule | Status | Rationale |
|------|--------|-----------|
| [Rule ID] | [Compliant/N/A] | [Brief rationale] |

## Generated Files

### Documentation
1. deployment-plan.md
2. monitoring-setup.md (if applicable)
3. cicd-pipeline.md (if applicable)
4. production-readiness-checklist.md (if applicable)
5. runbook.md (if applicable)
6. operations-summary.md

### Implementation Code
7. [IaC monitoring files] (if applicable)
8. [CI/CD pipeline config] (if applicable)
9. [Automation scripts] (if applicable)

## Estimated Additional Cost
| Item | Monthly Cost |
|------|-------------|
| [item] | [$ amount] |
| **Total** | **[$ amount]** |

## Next Steps
[Deployment execution steps or remaining items]
```

---

## Step 13: Present Completion Message

Present completion message in this structure:
1. **Completion Announcement** (mandatory): Always start with this:

```markdown
# 🚀 Operations Planning & Implementation Complete
```

2. **AI Summary** (optional): Provide structured bullet-point summary
   - Deployment strategy and target
   - Monitoring implementation summary (alarms, dashboards, log groups created)
   - CI/CD pipeline summary
   - Automation scripts generated
   - Production readiness status
   - Estimated cost impact
   - Key decisions and rationale
   - Extension rule compliance summary
   - Keep factual, no workflow instructions

3. **Formatted Workflow Message** (mandatory): Always end with this exact format:

```markdown
> **📋 AUTONOMOUS MODE: Skip review display, log completion in audit.md, proceed to next stage.**
> **📋 <u>**REVIEW REQUIRED:**</u>**  
> Please examine the operations artifacts at: `aidlc-docs/operations/`
>
> Documentation to review:
> - `deployment-plan.md` - Deployment strategy and steps
> - `monitoring-setup.md` - Metrics, alerts, and dashboards
> - `cicd-pipeline.md` - CI/CD pipeline design
> - `production-readiness-checklist.md` - Go/no-go checklist
> - `operations-summary.md` - Overall summary
>
> Implementation code to review:
> - IaC monitoring files (monitoring.tf, sns.tf, dashboard.tf, etc.)
> - CI/CD pipeline configuration (.github/workflows/, .gitlab-ci.yml, etc.)
> - Automation scripts (scripts/ops/)



> **🚀 <u>**WHAT'S NEXT?**</u>**
>
> **You may:**
>
> 🔧 **Request Changes** - Ask for modifications to the operations plan or generated code  
> ✅ **Complete AI-DLC** - Approve operations plan and finalize the AI-DLC workflow

---
```

---

## Step 14: Wait for Explicit Approval

- In autonomous mode: auto-approve and proceed. Do not wait for user to approve the operations plan and implementation code
- Approval must be clear and unambiguous
- If user requests changes, update the artifacts and repeat the approval process

---

## Step 15: Record Approval and Update Progress

- Log approval in audit.md with timestamp
- Record the user's approval response with timestamp
- Update `aidlc-docs/aidlc-state.md`:
  - Mark Operations stage as complete
  - Update current status to "Operations Complete"
- Present final AI-DLC completion message:

**IMPORTANT**: Dynamically construct the completion message based on stages that actually executed. Read `aidlc-docs/aidlc-state.md` to determine which stages were executed vs skipped, and only list executed stages in the summary.

```markdown
# ✅ AI-DLC Workflow Complete!

All phases have been completed:
- 🔵 **Inception**: [List only executed stages, e.g., "Requirements, design" or "Requirements, stories, design"] ✅
- 🟢 **Construction**: [List only executed stages, e.g., "Code generation, build & test"] ✅
- 🟡 **Operations**: [List only executed sub-stages, e.g., "Deployment, monitoring IaC, CI/CD pipeline, automation scripts, readiness"] ✅

All artifacts are available in `aidlc-docs/`.
Implementation code is ready for review and deployment.
```

---

## Step 16: Log Interaction

**MANDATORY**: Log the phase completion in `aidlc-docs/audit.md`:

```markdown
## Operations Stage
**Timestamp**: [ISO timestamp]
**Deployment Strategy**: [Strategy chosen]
**Monitoring**: [Configured/Skipped — number of alarms, dashboards]
**CI/CD Pipeline**: [Generated/Skipped — tool and stages]
**Automation Scripts**: [Generated/Skipped — list of scripts]
**Production Readiness**: [Status]
**Files Generated**:
### Documentation
- deployment-plan.md
- monitoring-setup.md
- cicd-pipeline.md
- production-readiness-checklist.md
- runbook.md
- operations-summary.md
### Implementation Code
- [list of IaC files]
- [list of pipeline files]
- [list of automation scripts]
**Estimated Additional Cost**: [$ amount/month]
**AI-DLC Status**: Complete

---
```
