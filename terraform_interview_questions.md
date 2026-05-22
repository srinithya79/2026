# 101 Terraform Interview Questions

## Top 20 Must-Know (Quick Cram)

- State and backends (locking, encryption): [Q1](#q1)–[Q3](#q3), [Q26](#q26)–[Q27](#q27)
- Providers, resources vs data sources: [Q4](#q4)–[Q5](#q5)
- Modules and outputs: [Q6](#q6), [Q18](#q18)
- for_each vs count: [Q7](#q7)
- Workspaces vs separate stacks: [Q8](#q8), [Q21](#q21)–[Q22](#q22)
- Importing existing infra: [Q9](#q9), [Q25](#q25)
- Secrets and sensitive handling: [Q10](#q10), [Q46](#q46)–[Q47](#q47)
- CI/CD pipeline and approvals: [Q11](#q11), [Q51](#q51)
- Policy as code (Sentinel/OPA): [Q12](#q12), [Q48](#q48)
- Lifecycle meta-args: [Q13](#q13), [Q69](#q69)–[Q70](#q70)
- Multi-account/multi-region patterns: [Q14](#q14), [Q42](#q42), [Q61](#q61)–[Q62](#q62)
- Drift detection and remediation: [Q15](#q15), [Q30](#q30), [Q52](#q52)
- Version pinning (Terraform and providers): [Q31](#q31)–[Q32](#q32)
- Safe upgrades and versioning practices: [Q33](#q33)–[Q35](#q35)
- Terragrunt for DRY orchestration: [Q88](#q88)
- Kubernetes and Helm providers (infra vs app): [Q63](#q63)
- VPC/networking via modules: [Q65](#q65)
- IAM policy best practices: [Q66](#q66)
- Cost control and FinOps (Infracost): [Q60](#q60)
- OpenTofu and licensing changes: [Q36](#q36), [Q98](#q98)–[Q99](#q99)


**1) What is state and why is it critical?**

```text
State maps config to real resources, enabling accurate plans and dependency tracking. It must be centralized, locked, encrypted, and backed up.
Example: S3 backend with DynamoDB lock prevents concurrent writes and protects secrets in state.
```
**2) Why not store state in Git?**

```text
State can contain secrets and ephemeral IDs; Git has no locking, causing race conditions and corruption risks. Use remote backends with locks and encryption.
```
**3) What is a backend and common choices?**

```text
A backend stores and locks state remotely. Common: AWS S3 + DynamoDB, GCS + locking, AzureRM, or Terraform Cloud/Enterprise remote.
Example: S3 bucket (SSE enabled) + DynamoDB lock table.
```
**4) What are providers?**

```text
Plug-ins that implement CRUD against service APIs (AWS, Azure, GCP, Kubernetes, Vault, etc.). Pin versions and configure auth per environment.
```
**5) Resource vs data source difference?**

```text
Resources create/manage objects; data sources read existing information. Use data sources for lookups (e.g., AMI IDs) without side effects.
```
**6) What are modules and why use them?**

```text
Reusable building blocks with inputs/outputs that enforce standards and DRY. Version modules, include examples, and publish to a registry for reuse.
```
**7) for_each vs count?**

```text
for_each uses stable keys for maps/sets (predictable addressing); count uses numeric indexes for identical instances. Prefer for_each for lifecycle-safe replacements.
Example: for_each on a map of subnet definitions instead of count.
```
**8) Workspaces—what and when?**

```text
Named state instances for the same config (e.g., dev/stage). Good for small variants; for strong isolation, use separate stacks/projects and permissions.
```
**9) How to import existing infra?**

```text
Use terraform import to map remote to a resource address, then author config to match. Tools like Terraformer can generate starter code; always verify and refactor.
```
**10) How to manage secrets?**

```text
Mark variables/outputs as sensitive, prefer Vault/cloud secret managers, avoid printing secrets in logs, and ensure backend encryption (SSE/CMEK). Never hardcode.
```
**11) Typical CI/CD pipeline?**

```text
fmt → init → validate → tflint/Checkov/Trivy → plan (artifact) → approval → apply. Add drift checks and post-apply smoke tests; gate applies via PRs.
```
**12) How to enforce policies?**

```text
Sentinel in TFC, or OPA/Conftest with CI run tasks to block non-compliant plans (e.g., unencrypted buckets, oversized instances, missing tags).
```
**13) Lifecycle meta-args?**

```text
create_before_destroy to minimize downtime, prevent_destroy to protect critical resources, ignore_changes to suppress churn from ephemeral fields.
Example: create_before_destroy for ASG launch template rotations.
```
**14) How to handle multi-account/multi-region?**

```text
Use provider aliases and assume-role profiles; structure modules per region/account; loop over regions cautiously and isolate state per account.
```
**15) How to detect and handle drift?**

```text
Schedule speculative plans in CI/TFC, alert on changes, reconcile via import/state mv, and document break-glass edits. Aim for frequent, small, reviewed applies.
```

## Core Concepts and HCL Basics
**16) What is Terraform in IaC?**

```text
Declarative desired state orchestration using providers to manage infra lifecycle consistently.
```
**17) How do you declare variables?**

```text
Use variable blocks with type/description/default; provide values via tfvars, env vars, or CLI -var.
```
**18) How do you pass outputs between modules?**

```text
Child modules expose outputs consumed by the parent; across stacks use remote state data sources.
```
**19) What are locals and why use them?**

```text
Computed values that simplify expressions, reduce repetition, and centralize transformations.
```
**20) Dynamic blocks—what’s the use case?**

```text
Programmatically generate nested blocks when schemas require repeated substructures.
```
**21) How do conditionals work in HCL?**

```text
Ternary condition ? a : b; combine with null to omit optional values.
```
**22) What are common HCL functions?**

```text
merge, concat, regex, jsonencode, yamldecode, flatten, distinct.
```
**23) How do you reference other resources?**

```text
resource.type.name.attribute; qualify with module paths for nested modules.
```
**24) What do fmt and validate do?**

```text
fmt standardizes formatting; validate checks configuration structure before plan.
```
**25) Recommended file layout?**

```text
main.tf, variables.tf, outputs.tf, providers.tf; modules/ for reuse and examples.
```

## State, Backends, and Workspaces
**26) How do you implement state locking?**

```text
Use backend-native locks (e.g., DynamoDB for S3). Prevents concurrent writes and corruption.
```
**27) How do you ensure state encryption?**

```text
Enable SSE/CMEK in cloud backends; Terraform Cloud encrypts state by default.
```
**28) How to inspect/move/remove state items?**

```text
terraform state list/show; terraform state mv to rename; terraform state rm to remove orphaned entries.
```
**29) What happens on apply failures/tainted resources?**

```text
Failed resources may be tainted, prompting replacement on next apply unless resolved.
```
**30) How do you detect drift at scale?**

```text
Schedule speculative plans in CI/TFC and alert on unexpected changes.
```

## Language and Versioning
**31) How do you pin provider versions?**

```text
Use required_providers in the terraform block with version constraints.
```
**32) How do you pin Terraform CLI version?**

```text
Set required_version in the terraform block.
```
**33) How do modules handle versioning?**

```text
Constrain module versions in source; follow semver and changelogs.
```
**34) How do you upgrade providers/modules safely?**

```text
terraform init -upgrade, review breaking changes, and test in non-prod first.
```
**35) What’s notable in Terraform 1.x?**

```text
Stable language, richer functions, improved CLI, better ecosystem maturity.
```
**36) What is OpenTofu?**

```text
Community fork of Terraform with similar CLI; open governance and compatibility focus.
```
**37) What is CDK for Terraform (CDKTF)?**

```text
Use TypeScript/Python/etc to synthesize Terraform configs programmatically.
```

## Patterns and Design
**38) How do you structure multi-environment projects?**

```text
Separate stacks per environment with shared modules; drive variants via tfvars or Terragrunt.
```
**39) How do you share values across stacks?**

```text
Use remote state data sources, TFC workspace outputs, or service discovery.
```
**40) How to avoid duplicate security group rules?**

```text
A reusable module with for_each and structured inputs; use dynamic blocks for nested rules.
```
**41) How do you model explicit dependencies?**

```text
depends_on for non-reference ordering; otherwise rely on implicit references.
```
**42) How do you bootstrap multi-account setups?**

```text
Landing zone modules, provider aliases, assume-role patterns, and per-account state isolation.
```
**43) When should you use provisioners?**

```text
Rarely. Prefer cloud-init/user data. Use null_resource only when unavoidable.
```
**44) How do you pass files/content?**

```text
file() for raw file, templatefile() for templating, base64encode for binary/user data.
```
**45) How do you build and reference artifacts (e.g., Lambda zips)?**

```text
data.archive_file during apply or pre-build in CI and reference the artifact path.
```

## Security and Secrets
**46) How do you handle secrets?**

```text
Use sensitive variables/outputs, secret managers (Vault/cloud), and backend encryption; avoid exposing secrets in logs.
```
**47) What does sensitive=true actually do?**

```text
Masks values in CLI/UI but they still exist in state; rely on backend encryption and strict access controls.
```
**48) How do you enforce security/compliance policies?**

```text
Sentinel (TFC) or OPA/Conftest with CI and run tasks to block non-compliant plans.
```
**49) Which static analysis tools are common?**

```text
tflint for linting, Trivy/tfsec and Checkov for security/compliance checks.
```

## Testing and CI/CD
**50) How do you test Terraform?**

```text
Terratest (Go), kitchen-terraform, and post-apply smoke tests; always run validate/plan in CI.
```
**51) How do you implement plan approvals?**

```text
TFC manual gates or CI approvals before apply, with plan artifacts attached to PRs.
```
**52) How do you detect drift continuously?**

```text
Scheduled speculative plans compare against last apply; notify on unexpected changes.
```
**53) How do you generate docs?**

```text
Use terraform-docs for modules to auto-generate inputs/outputs/usage.
```

## Performance and Scale
**54) How do you speed up large applies?**

```text
Split stacks, reduce unnecessary dependencies, and tune parallelism.
```
**55) How do you control parallelism?**

```text
Use -parallelism=N to limit concurrent operations for stability.
```
**56) How do you avoid API throttling?**

```text
Backoff/retries, batching operations, and respecting provider rate limits.
```

## Collaboration and Governance
**57) How do you avoid state conflicts?**

```text
Remote backends with locking, isolated workspaces/stacks, and PR-based workflows.
```
**58) How do you track who changed infra?**

```text
VCS commit history, TFC run logs, and cloud audit trails (CloudTrail/Azure Activity/GCP Audit).
```
**59) How do you enforce naming/tagging standards?**

```text
Module-required inputs, policy checks (OPA/Sentinel), and pre-commit hooks.
```
**60) How do you control costs?**

```text
Budgets, policy checks on instance types/sizes, and tag-based reporting/alerts.
```

## Providers and Cloud-Specific
**61) How do you authenticate providers locally?**

```text
Env vars, profiles, and CLI auth flows; never hardcode secrets in configs.
```
**62) How do you assume roles in AWS?**

```text
Configure provider assume_role or use profiles; leverage STS with session names.
```
**63) How do you manage Kubernetes with Terraform?**

```text
Use kubernetes provider for infra-level objects; avoid heavy application lifecycle operations.
```
**64) How do you manage DNS/certificates?**

```text
Use cloud DNS providers and acme providers; validate via data sources.
```
**65) How do you handle VPC/networking?**

```text
Adopt official/community VPC modules with structured inputs for subnets and routing.
```
**66) How do you attach IAM policies safely?**

```text
Use jsonencode for inline policies, prefer managed policies, and apply least privilege.
```

## Advanced Topics
**67) What is resource addressing?**

```text
The unique path (module.module.resource) identifying resources in state; used for imports and mv/rm.
```
**68) What is replace_triggered_by?**

```text
Meta-arg that forces replacement when specific attributes change.
```
**69) What is the lifecycle meta-argument?**

```text
Controls create_before_destroy, prevent_destroy, and ignore_changes behaviors.
```
**70) How do you handle ephemeral attribute churn?**

```text
Use ignore_changes on volatile fields to avoid unnecessary replacements.
```
**71) How do you manage ordering with data sources?**

```text
Add depends_on to data sources when evaluation order matters.
```
**72) How do you use external data?**

```text
external data source or local-exec; prefer deterministic and versioned inputs.
```
**73) How do you serialize complex objects?**

```text
Use jsonencode/yamldecode (and toset/tomap) to pass structured data safely.
```

## Troubleshooting
**74) How do you debug plans?**

```text
terraform show -json, TF_LOG for detailed logs, and terraform graph for dependency visualization.
```
**75) Provider not found—what to check?**

```text
required_providers, network connectivity, and run terraform init.
```
**76) “Provider configuration not present”—why?**

```text
Missing aliased provider in child module; pass providers map explicitly.
```
**77) How do you resolve dependency cycles?**

```text
Break references, use data sources, and restructure module boundaries.
```
**78) How do you recover from a bad state?**

```text
state rm/mv, import resources, targeted apply, and restore from backend backups as needed.
```

## Commands and CLI
**79) What does init do?**

```text
Downloads providers/modules and configures the backend.
```
**80) What does plan do?**

```text
Produces an execution plan showing intended changes without applying.
```
**81) What does apply do?**

```text
Executes the plan; can be run with -auto-approve for non-interactive runs.
```
**82) What does destroy do?**

```text
Removes all managed resources in the current state.
```
**83) What’s refresh behavior today?**

```text
Standalone refresh is deprecated; refresh occurs during plan/apply.
```
**84) What do fmt and validate do?**

```text
fmt formats code; validate checks config correctness before plan.
```
**85) How do you target specific resources?**

```text
Use -target during plan/apply sparingly for break-glass scenarios.
```
**86) How to run non-interactively?**

```text
-auto-approve and provide inputs via env vars or tfvars.
```
**87) How do you view dependency graphs?**

```text
terraform graph piped to dot to render visualization.
```

## Ecosystem and Alternatives
**88) What is Terragrunt?**

```text
Wrapper offering DRY, dependency wiring, remote state patterns, and hierarchical configs.
```
**89) Packer vs Terraform?**

```text
Packer builds machine images; Terraform provisions infrastructure using those images.
```
**90) How do pre-commit hooks help?**

```text
Enforce fmt, validate, lint/security on each commit to keep quality consistent.
```
**91) How do you publish modules?**

```text
Semantic versioning, good docs/examples, registry metadata, and tests.
```
**92) What is a registry source address?**

```text
namespace/name/provider with version constraints.
```

## Cost, Compliance, and Ops
**93) How do you restrict instance types?**

```text
Policies (OPA/Sentinel), CI checks, and module validation rules.
```
**94) How do you ensure encryption?**

```text
Backend SSE/CMEK, encrypted volumes/buckets, and KMS/CMEK usage in resources.
```
**95) How to ensure tagging consistency?**

```text
Module-required tag maps and policy checks; fail CI if tags are missing.
```
**96) What’s your rollback strategy?**

```text
Re-apply previous config versions, maintain state backups, and plan before changes.
```
**97) How to handle break-glass changes?**

```text
Document manual edits, reconcile via import/plan, review, and restore desired state.
```

## New and Noteworthy (2024–2025)
**98) What’s the impact of licensing changes?**

```text
Emergence of OpenTofu; teams reassess tooling alignment and ecosystem support.
```
**99) How does OpenTofu differ today?**

```text
Community-led with similar CLI; verify provider compatibility for workloads.
```
**100) What are Terraform Cloud run tasks?**

```text
Pre/post run integrations to enforce checks (security/tests/compliance) around runs.
```
**101) How do you approach multi-tenant orgs?**

```text
Workspace-per-service, scoped teams/policies, VCS integration, variable sets, and remote state sharing.
```

---

## Snippets Appendix (copy-paste friendly)

1) Backend (S3 + DynamoDB locking)

```hcl
terraform {
  backend "s3" {
    bucket         = "my-tf-state-bucket"
    key            = "envs/prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "tf-locks"
  }
}
```

2) Required providers and CLI version

```hcl
terraform {
  required_version = ">= 1.5.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

3) Provider alias and assume_role (multi-account)

```hcl
provider "aws" {
  region = "us-east-1"
}

provider "aws" {
  alias  = "prod"
  region = "us-east-1"
  assume_role {
    role_arn     = "arn:aws:iam::123456789012:role/Admin"
    session_name = "tf-session"
  }
}
```

4) for_each map pattern

```hcl
variable "rules" {
  type = map(object({ port = number, cidr = string }))
}

resource "aws_security_group_rule" "ingress" {
  for_each          = var.rules
  type              = "ingress"
  from_port         = each.value.port
  to_port           = each.value.port
  protocol          = "tcp"
  cidr_blocks       = [each.value.cidr]
  security_group_id = aws_security_group.app.id
}
```

5) Dynamic block example

```hcl
resource "aws_security_group" "app" {
  name = "app-sg"

  dynamic "ingress" {
    for_each = var.ingress_list
    content {
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = "tcp"
      cidr_blocks = ingress.value.cidrs
    }
  }
}
```

6) Lifecycle meta-arguments

```hcl
resource "aws_launch_template" "app" {
  name = "app-lt"

  lifecycle {
    create_before_destroy = true
    ignore_changes        = [user_data]
  }
}
```

7) terraform_remote_state (cross-stack outputs)

```hcl
data "terraform_remote_state" "network" {
  backend = "s3"
  config = {
    bucket = "my-tf-state-bucket"
    key    = "envs/prod/network.tfstate"
    region = "us-east-1"
  }
}

output "vpc_id" {
  value = data.terraform_remote_state.network.outputs.vpc_id
}
```

8) Default tags and sensitive variable

```hcl
provider "aws" {
  region = var.region
  default_tags {
    tags = {
      Env     = var.env
      Project = "my-app"
    }
  }
}

variable "db_password" {
  type      = string
  sensitive = true
}
```
