
# ══════════════════════════════════════════════════════════════
#   TERRAFORM ASSOCIATE EXAM – ULTIMATE CRAM SHEET
#   Generated from ExamTopics (v1, v003, v004)
# ══════════════════════════════════════════════════════════════

---

# 🌳 TERRAFORM DECISION TREES

## STATE & DRIFT DECISION TREE

```
Need to reconcile infrastructure?
│
├─ Infrastructure changed manually (outside Terraform)?
│  └─ YES → That is DRIFT
│        ├─ Preview drift only  → terraform plan -refresh-only
│        └─ Accept drift        → terraform apply -refresh-only
│              (updates state to match reality, no config changes)
│
├─ Remove resource from config, but resource was manually destroyed?
│  └─ Remove .tf definition + terraform apply -refresh-only
│
├─ Resource removed from .tf config, existing resource still alive?
│  └─ terraform apply → Terraform DESTROYS the real resource
│
├─ Force recreate a resource?
│  ├─ Old (deprecated)  → terraform taint <resource>
│  └─ Modern (v0.15.2+) → terraform apply -replace=<resource>
│
└─ State file stale/locked?
   ├─ Automatic unlock failed → terraform force-unlock <lock-id>
   └─ legacy refresh (avoid)  → terraform refresh
```

---

## BACKEND DECISION TREE

```
Need to store/share state?
│
├─ Local only (default)?
│  └─ terraform.tfstate in working dir
│
├─ Remote shared state (team)?
│  ├─ S3  → needs DynamoDB for locking
│  ├─ Consul → has built-in locking
│  └─ HCP Terraform → built-in locking + remote execution
│
├─ Prevent concurrent state changes?
│  └─ Configure STATE LOCKING on backend (S3+DynamoDB, Consul, HCP)
│
├─ Multiple environments from ONE backend config?
│  ├─ workspaces.name   → maps to ONE workspace
│  └─ workspaces.prefix → maps to MULTIPLE workspaces
│
├─ Migrate backend (local → S3)?
│  └─ Update backend.tf + terraform init  (will prompt to migrate)
│     Modern: terraform init --migrate-state
│
└─ Sensitive data in state?
   └─ Use encrypted backend (NOT local plaintext)
```

---

## PROVIDER & MODULE DECISION TREE

```
Working with providers?
│
├─ First time / new provider added?
│  └─ terraform init  (downloads to .terraform/providers/)
│
├─ Upgrade to latest provider versions?
│  └─ terraform init -upgrade
│     (respects version constraints, updates .terraform.lock.hcl)
│
├─ Lock file (.terraform.lock.hcl) exists?
│  └─ Terraform uses version FROM lock file (not latest)
│
├─ Provider from internet vs offline?
│  ├─ Default → registry.terraform.io
│  └─ Air-gapped → filesystem/network mirror
│
├─ Multiple configs for same provider?
│  └─ Use provider ALIAS
│
├─ What's stored in .terraform/ dir?
│  └─ Providers AND Modules (not lock file, not state)
│
└─ Plugin-based architecture?
   ├─ Providers are plugins (separate binaries)
   ├─ Can write custom provider for any API
   ├─ Provider types: Official / Partner / Community
   └─ Provider = first part of resource type (aws_vpc → aws)
```

---

## WORKSPACE DECISION TREE

```
Need multiple environments?
│
├─ CLI Workspaces?
│  ├─ terraform workspace new <name>
│  ├─ terraform workspace list
│  ├─ terraform workspace select <name>
│  └─ Each workspace has its own state file
│
├─ HCP Terraform Workspaces vs CLI Workspaces?
│  ├─ CLI workspace → local directory state isolation
│  └─ HCP workspace → full remote env with variables, policies, VCS
│
├─ Feature ONLY in HCP (not CLI)?
│  └─ Secure variable storage
│
└─ One cloud block = one org. Can define MULTIPLE workspaces
   └─ One cloud block per configuration (cannot configure multiple)
```

---

## LIFECYCLE DECISION TREE

```
Need to control resource lifecycle?
│
├─ Prevent accidental deletion?
│  └─ lifecycle { prevent_destroy = true }
│
├─ Create before destroy (zero-downtime)?
│  └─ lifecycle { create_before_destroy = true }
│
├─ Ignore changes (config drift tolerated)?
│  └─ lifecycle { ignore_changes = [<attr>] }
│
├─ Replace resource on next apply?
│  └─ lifecycle { replace_triggered_by = [...] }
│
└─ Check block assertion fails?
   └─ WARNING only (non-blocking) – operation continues
```

---

## DEPENDENCY DECISION TREE

```
Managing resource dependencies?
│
├─ Terraform detects dependencies automatically?
│  └─ YES – via attribute references (implicit)
│
├─ Explicit dependency needed (no attribute reference)?
│  └─ depends_on = [<resource>]
│
├─ count vs for_each?
│  ├─ count  → creates LIST  → reference with [index], use splat [*]
│  └─ for_each → creates MAP → reference with ["key"], NO splat [*]
│
└─ Dynamic nested blocks?
   └─ Use dynamic blocks (not count arguments)
```

---

# ⚡ COMMANDS QUICK REFERENCE

## Core Workflow
| Command | Purpose |
|---|---|
| `terraform init` | FIRST command always – downloads providers/modules, init backend |
| `terraform plan` | Preview changes (read-only, does not change state) |
| `terraform apply` | Execute changes (refreshes state by default) |
| `terraform destroy` | Destroy all managed resources |

## Validate & Format
| Command | Purpose |
|---|---|
| `terraform validate` | Syntax + internal consistency check (needs init, no API calls) |
| `terraform fmt` | Format code (tabs vs spaces – validate does NOT catch this) |

## State Operations
| Command | Purpose |
|---|---|
| `terraform state list` | List all resources in state |
| `terraform state show <res>` | Show attributes of a resource |
| `terraform state rm <res>` | Remove resource from state (keeps real infra) |
| `terraform state mv` | Move resource in state |

## Refresh / Drift
| Command | Purpose |
|---|---|
| `terraform plan -refresh-only` | Preview state refresh (no infra changes) |
| `terraform apply -refresh-only` | Apply state refresh (updates state file) |
| `terraform refresh` | Legacy – deprecated behavior |

## Taint / Replace
| Command | Purpose |
|---|---|
| `terraform taint <res>` | Mark for recreate – **DEPRECATED** |
| `terraform apply -replace=<res>` | Modern replace (v0.15.2+) |

## Import
| Command | Purpose |
|---|---|
| `terraform import <address> <id>` | Import existing resource into state |

> `terraform import` requires: **resource address** + **resource ID**  
> You must ALSO write the `.tf` config manually (import doesn't generate it)

## Destroy Preview
| Command | Purpose |
|---|---|
| `terraform plan -destroy` | Preview destroy plan |
| `terraform destroy` | Shows resources to delete before prompting |

## Locking
| Command | Purpose |
|---|---|
| `terraform force-unlock <id>` | Force unlock state – use ONLY when auto-unlock fails |

## Debugging
| Command | Purpose |
|---|---|
| `TF_LOG=DEBUG` | Enable debug logging |
| `TF_LOG=TRACE` | Most verbose – shows provider load paths |
| `terraform console` | Interactive expression evaluation |
| `terraform providers` | Show provider requirements |
| `terraform show` | Show current state or plan |
| `terraform output` | Show output values |
| `terraform output <name>` | Show specific output |

## Workspace
| Command | Purpose |
|---|---|
| `terraform workspace new <name>` | Create workspace |
| `terraform workspace list` | List workspaces |
| `terraform workspace select <name>` | Switch workspace |

## HCP Terraform
| Command | Purpose |
|---|---|
| `terraform login` | Authenticate with HCP Terraform |

---

# 🚨 EXAM TRAPS

## State & Drift
```
❌ TRAP: terraform.tfstate ALWAYS matches current infrastructure
→ FALSE – drift occurs when changes happen outside Terraform
   State = snapshot of LAST KNOWN state, not real-time

❌ TRAP: terraform plan always == terraform apply execution plan
→ FALSE – teammate manual change between plan & apply = different result
   Apply refreshes state again before executing

❌ TRAP: terraform plan -refresh-only UPDATES the state file
→ FALSE – plan is preview only; state updated only with apply -refresh-only

❌ TRAP: Changing resource in Azure Console updates state file
→ FALSE – Console changes do NOT update state file
   State updates only during next plan/apply/refresh

❌ TRAP: Remove resource from .tf config = Terraform removes from state only
→ FALSE – Terraform DESTROYS the real resource on next apply
```

## Backend & Workspaces
```
❌ TRAP: One remote backend config = one remote workspace
→ FALSE – use workspaces.prefix for MULTIPLE workspaces

❌ TRAP: All standard backends support state locking + remote operations
→ FALSE – only some backends support locking; only HCP Terraform is "enhanced"

❌ TRAP: State file is secure by default
→ FALSE – local state = plaintext JSON; use encrypted backend

❌ TRAP: One cloud block maps to exactly one HCP workspace
→ FALSE – cloud block → one org, can configure multiple workspaces

❌ TRAP: You cannot change backend after first terraform apply
→ FALSE – it is OPTIONAL but possible; run terraform init to migrate
```

## Providers & Plugins
```
❌ TRAP: Provider configuration block required in every config
→ FALSE – block can be omitted; Terraform assumes empty default
   Provider is required; provider BLOCK is not always required

❌ TRAP: Providers are always installed from the internet
→ FALSE – air-gapped installs use filesystem/network mirrors

❌ TRAP: Provider versions from state file
→ FALSE – Terraform uses .terraform.lock.hcl (lock file), NOT state

❌ TRAP: terraform init -upgrade uses ANY latest version
→ FALSE – still respects version constraints in configuration
```

## Variables & Expressions
```
❌ TRAP: Variable declarations require type, default, or description
→ FALSE – all are optional; variable "x" {} is valid (empty block)

❌ TRAP: for_each resources can use splat [*] expression
→ FALSE – splat works with count (list); for_each = map → use for expressions
   count → list → splat [*] ✅
   for_each → map → splat [*] ❌ (use for expression instead)
```

## Terraform validate
```
❌ TRAP: terraform validate uses provider APIs
→ FALSE – validate is offline; checks syntax only, no API calls

❌ TRAP: terraform validate catches tab indentation errors
→ FALSE – tabs are allowed; terraform fmt fixes indentation, not validate

✅ validate DOES catch: missing variable block declarations
```

## Outputs
```
❌ TRAP: terraform apply prints outputs from child modules
→ FALSE – only ROOT module outputs are printed; child module outputs are not
```

## Provisioners
```
local-exec  → runs on machine running Terraform (NOT on resource)
remote-exec → runs on the resource created by Terraform
file        → copies files to the resource
```

## Immutable Infrastructure
```
Immutable infra advantage = Less complex upgrades
(replace entirely instead of modifying in place)
NOT: automatic, in-place, or quicker
```

## Dependency
```
❌ TRAP: Terraform ONLY manages dependencies via depends_on
→ FALSE – Terraform handles most dependencies implicitly via references
   depends_on needed only when no attribute reference exists
```

---

# 🎯 KEYWORD PATTERN RECOGNITION

## "If question mentions..." → "Answer is..."

| Keyword in Question | → Answer |
|---|---|
| "manually changed infrastructure" | → Drift |
| "sensitive values / secrets in state" | → Use encrypted backend |
| "up-to-date / latest provider versions" | → `terraform init -upgrade` |
| "multiple remote workspaces from one config" | → `workspaces.prefix` |
| "single remote workspace" | → `workspaces.name` |
| "resource recreated next apply" | → `taint` / `-replace` |
| "rerun local-exec provisioner" | → `taint` or `-replace` (taint if old exam) |
| "force recreate modern way" | → `terraform apply -replace=` |
| "sync state without infrastructure change" | → `terraform apply -refresh-only` |
| "preview state sync" | → `terraform plan -refresh-only` |
| "prevent two runs modifying same state" | → State locking |
| "state lock stuck / automatic unlock failed" | → `force-unlock` |
| "migrate state to new backend" | → `terraform init` |
| "access denied error debugging" | → `TF_LOG=DEBUG` |
| "trace provider load paths" | → `TF_LOG=TRACE` |
| "experiment with expressions" | → `terraform console` |
| "first command in new directory" | → `terraform init` |
| "import existing resources" | → `terraform import` + write config |
| "no outputs defined, find IP" | → `terraform state list` + `state show` |
| "check configuration syntax" | → `terraform validate` |
| "format code" | → `terraform fmt` |
| "show resources to be deleted" | → `terraform plan -destroy` or `terraform destroy` |
| "run on resource (not local)" | → `remote-exec` provisioner |
| "run on Terraform machine" | → `local-exec` provisioner |
| "copy file to resource" | → `file` provisioner |
| "private module source" | → Private GitHub repo OR internal VCS |
| "multiple nested blocks from variable" | → `dynamic` block |
| "second instance of count resource" | → `resource[1]` (0-indexed) |
| "provider for aws_vpc" | → `aws` (prefix before first `_`) |
| "lock file created when" | → `terraform init` |
| "what's stored in .terraform/" | → Providers and modules |
| "health assessment in HCP" | → Drift detection + Check block validation |
| "HCP Terraform vs S3/Consul difference" | → HCP can EXECUTE runs (enhanced backend) |
| "available only in HCP, not CLI" | → Secure variable storage |
| "check block assertion fails" | → Warning only (non-blocking) |
| "can only have one cloud block" | → TRUE (only one per config) |
| "write Terraform config first time workflow" | → write → init → plan → apply |
| "advantage of immutable infra" | → Less complex upgrades |

---

# 🏗️ TOPIC SECTIONS

## STATE
```
State file = snapshot of last known infrastructure
Location: terraform.tfstate (local) or remote backend

terraform.tfstate          = current state
terraform.tfstate.backup   = previous state

State stores:
  - Resource metadata
  - Attribute values (including SECRETS in plaintext if local!)
  - Resource dependencies

State does NOT store:
  - Provider version (that's in lock file)
  - Real-time infra status (drift not auto-detected)
```

## BACKEND
```
Standard backends:  S3, Consul, GCS, Azure Blob
  → store state only
  → some support locking (S3+DynamoDB, Consul)

Enhanced backend:   HCP Terraform / Terraform Cloud
  → stores state + EXECUTES operations remotely
  → has built-in locking
  → has Sentinel policy checks
  → secure variable storage
  → VCS integration

Local backend (default):
  → terraform.tfstate in working dir
  → no locking
  → plaintext (insecure for secrets)
```

## WORKSPACE
```
CLI workspace:
  - Separate state per workspace
  - Same config, different state
  - terraform.workspace = current workspace name
  - terraform workspace <new|list|select|delete>

Remote workspace (HCP):
  - workspaces.name   → SINGLE remote workspace
  - workspaces.prefix → MULTIPLE remote workspaces
    e.g., prefix = "networking-" → networking-dev, networking-prod

cloud block (modern) vs remote block (legacy):
  - cloud block = connects to HCP Terraform
  - One cloud block per config
  - cloud block → one org, multiple workspaces possible
```

## PROVIDERS
```
Types:
  - Official  → maintained by HashiCorp
  - Partner   → maintained by cloud vendors
  - Community → maintained by individuals/community

ALL of the above are TRUE (exam trick: "which is NOT true" → None of above)

Provider architecture:
  - Plugins (separate binaries from Terraform core)
  - Communicate via RPC
  - Downloaded to .terraform/providers/
  - Can write custom providers for any API

Version pinning:
  .terraform.lock.hcl = dependency lock file
  Created/updated by: terraform init
  Priority: lock file > version constraints
  To upgrade: terraform init -upgrade

Provider reference in code:
  resource "aws_vpc" "main" {} → provider = aws
  Outside required_providers → refer by LOCAL name only
```

## MODULES
```
Module sources (private):
  - Private GitHub repo ✅
  - Internal VCS platform ✅
  - Public Terraform Registry ❌ (not private)
  - Public GitHub ❌ (not private)

terraform import:
  → syntax: terraform import <address> <resource_id>
  → requires: resource address + resource ID
  → does NOT generate config (write manually!)
  → use case: bring existing resource under TF management

Module outputs:
  → terraform apply only prints ROOT module outputs
  → child module outputs not displayed automatically
```

## COMMANDS (Anti-confusion)
```
terraform show       → show state or plan file content
terraform output     → show defined output values
terraform state show → show state for specific resource
terraform validate   → syntax check (no API, needs init)
terraform fmt        → format HCL code
terraform providers  → show provider requirements
terraform console    → interactive expression REPL

NOT REAL commands:
  terraform push         → deprecated
  terraform env          → deprecated (use workspace)
  terraform import-gcp   → does not exist
  terraform show -destroy → does not exist
  TF_VAR_log=TRACE       → wrong; use TF_LOG=TRACE
  TF_LOG=PATH            → wrong; use TF_LOG_PATH for log file path
```

## HCP TERRAFORM
```
Key features:
  - Remote execution (plan/apply on HCP VMs)
  - Secure variable storage (ONLY in HCP, not CLI)
  - State storage + locking
  - Sentinel policy checks
  - VCS integration (GitHub, GitLab, etc.)
  - Team management
  - Health assessments:
      ✅ Resource drift detection
      ✅ Check block validation
      ❌ NOT: Sentinel policy checks during health assessment
      ❌ NOT: Estimate infrastructure cost

HCP free tier exists (NOT only for paying customers)
```

## SECURITY
```
⚠️  State file stores secrets in PLAINTEXT by default
⚠️  Sensitive outputs still stored in state

Solutions:
  - Encrypted backend (S3+SSE, Azure Blob encryption, etc.)
  - HCP Terraform (encrypted state at rest)
  - DO NOT: delete state, edit state manually, store in secrets.tfvars

Best practice → Protect state = Encrypt backend
```

## DRIFT
```
Drift = difference between real infra and state file
Cause = changes made outside Terraform (manual, other tools)

Detection:
  terraform plan         → shows drift as changes to apply
  terraform plan -refresh-only → shows drift only (no config changes)

Resolution options:
  1. Correct the drift: terraform apply (overwrite manual change with config)
  2. Accept the drift:  terraform apply -refresh-only (update state to match reality)

KEY: State file does NOT auto-update on external changes
     Updates happen on next: plan, apply, or refresh
```

## LIFECYCLE
```
lifecycle {
  create_before_destroy = true   # zero-downtime replacement
  prevent_destroy       = true   # block destroy
  ignore_changes        = [attr] # tolerate external changes
  replace_triggered_by  = [...]  # force replace when dep changes
}

Taint (deprecated in 0.15.2):
  terraform taint <resource>
  → replaced by: terraform apply -replace=<resource>

check block:
  → health assertion (non-blocking)
  → failure = warning, NOT error
  → operation continues
```

## VARIABLES
```
Variable declaration:
  variable "name" {}   → ALL arguments optional
  Required args: NONE
  Optional args: type, default, description, validation, sensitive

Variable precedence (highest → lowest):
  1. -var or -var-file CLI flags
  2. *.auto.tfvars / *.auto.tfvars.json
  3. terraform.tfvars
  4. Environment variables TF_VAR_*
  5. Default value in variable block

Splat expressions:
  count  → list → aws_instance.web[*].id ✅
  for_each → map → aws_instance.web[*].id ❌ (use for expression)

For expressions:
  List: [ for o in var.list : o.id ]
  Map:  { for o in var.list : o.name => o.id }
```

## OUTPUTS
```
output "name" {
  value       = resource.type.name.attr
  description = "..."
  sensitive   = true   # hides from CLI output, still in state
}

terraform output          → show all root outputs
terraform output <name>   → show specific output
terraform state show      → show raw state values

Child module outputs → NOT printed by terraform apply
```

## FUNCTIONS
```
Common exam functions:
  length()     → count items
  toset()      → convert list to set
  flatten()    → flatten nested lists
  merge()      → merge maps
  lookup()     → map lookup with default
  element()    → get item by index
  join()       → join list to string
  split()      → split string to list
  format()     → string formatting
  coalesce()   → first non-null value
```

## DEPENDENCIES
```
Implicit dependency (automatic):
  resource "A" depends on resource "B".attr → TF detects it

Explicit dependency (manual):
  resource "A" {
    depends_on = [resource.B]
  }
  Use when: dependency cannot be expressed via attribute reference

Provider dependency:
  depends_on in module blocks also works
```

## PROVISIONERS
```
local-exec:
  → runs command on Terraform machine (where TF is running)
  → use for: Ansible, scripts triggered locally

remote-exec:
  → runs command ON the resource (SSH/WinRM)
  → use for: bootstrapping remote server

file:
  → copies files from local to remote resource

⚠️  Provisioners are last resort – prefer provider resources
⚠️  Terraform recommends AVOIDING provisioners when possible

To re-run provisioner:
  Modern: terraform apply -replace=<resource>
  Legacy: terraform taint <resource> → terraform apply
```

## IMMUTABLE INFRASTRUCTURE
```
Immutable = replace entire resource instead of modifying
Advantage = Less complex upgrades
  ✅ Less complex
  ❌ NOT: Automatic / In-place / Quicker

Terraform encourages immutable via:
  create_before_destroy lifecycle
  -replace flag
```

---

# ⚙️ TERRAFORM WORKFLOW (CANONICAL)

```
New infrastructure workflow:
  1. Write Terraform configuration (.tf files)
  2. terraform init       (setup providers, modules, backend)
  3. terraform plan       (preview – optional but recommended)
  4. terraform apply      (execute)
  5. terraform destroy    (teardown when done)

Import existing infrastructure:
  1. Write matching .tf configuration
  2. terraform import <address> <id>
  3. Verify with terraform plan (should show no changes)

Migrate backend:
  1. Update/add backend block in configuration
  2. terraform init       (prompts to copy existing state)
```

---

# 🔑 QUICK MEMORY

```
init    → initialize (providers, modules, backend, lock file)
plan    → preview (read-only, refreshes state)
apply   → execute (refreshes state, makes changes)
destroy → remove all

fmt      → formatting (tabs→spaces etc.)
validate → syntax check (no API, needs init, not validate tabs)
show     → display state or plan
output   → display outputs
console  → expression REPL

state list → list resources
state show → show resource attributes
state rm   → remove from state (not from cloud)

taint    → DEPRECATED → use -replace
refresh  → DEPRECATED behavior → use -refresh-only

import   → bring existing into state (write config first!)
workspace → manage multiple state environments
```

---

# 🎯 MOST-TESTED PATTERNS (HIGH UPVOTE QUESTIONS)

```
1. tfstate ALWAYS matches infra?       → FALSE (drift possible)
2. One remote backend = one workspace? → FALSE (prefix = multiple)
3. Provider block required everywhere? → FALSE (can be omitted)
4. Immutable infra advantage?          → Less complex upgrades
5. Rerun provisioner?                  → taint/replace then apply
6. First command ever?                 → terraform init
7. Secure secrets in state?            → Encrypted backend
8. Upgrade providers?                  → terraform init -upgrade
9. Access denied debugging?            → TF_LOG=DEBUG
10. for_each + splat?                  → CANNOT use splat [*]
11. Variable required args?            → NONE (all optional)
12. Remote backend vs S3?              → HCP executes runs (enhanced)
13. Prevent concurrent state changes?  → State locking
14. force-unlock when?                 → Only when auto-unlock fails
15. Lock file created when?            → terraform init
16. What in .terraform/ dir?           → Providers and modules
17. validate uses provider APIs?       → FALSE (offline check)
18. apply prints child module outputs? → FALSE (root only)
19. check block failure blocks apply?  → FALSE (warning only)
20. Multiple cloud blocks allowed?     → FALSE (one per config)
21. depends_on only way to depend?     → FALSE (implicit exists)
22. Providers from internet only?      → FALSE (mirrors exist)
23. State checks provider version?     → FALSE (lock file does)
24. Terraform validate before init?    → FALSE (need init first)
25. Remove config → what happens?      → Terraform DESTROYS real resource
```

---

# 📐 REFERENCE SYNTAX

```hcl
# Backend (S3 with locking)
terraform {
  backend "s3" {
    bucket         = "my-bucket"
    key            = "terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"  # locking
  }
}

# Remote backend workspace modes
terraform {
  backend "remote" {
    organization = "my-org"
    workspaces {
      name   = "prod"         # single workspace
      # prefix = "app-"       # multiple workspaces
    }
  }
}

# HCP Terraform (cloud block)
terraform {
  cloud {
    organization = "my-org"
    workspaces {
      name = "prod"
    }
  }
}

# Lifecycle
resource "aws_instance" "web" {
  lifecycle {
    create_before_destroy = true
    prevent_destroy       = true
    ignore_changes        = [tags]
  }
}

# Dynamic block
resource "aws_security_group" "sg" {
  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port = ingress.value.port
      to_port   = ingress.value.port
      protocol  = ingress.value.protocol
    }
  }
}

# for_each vs count
resource "aws_instance" "count_ex" {
  count = 3
  # reference: aws_instance.count_ex[0], [1], [2]
  # splat:     aws_instance.count_ex[*].id  ✅
}

resource "aws_instance" "fe_ex" {
  for_each = { "web" = "t2.micro", "api" = "t2.small" }
  instance_type = each.value
  # reference: aws_instance.fe_ex["web"]
  # splat: ❌ use for expression instead
}

# Import (required: address + ID)
# terraform import aws_instance.web i-1234567890abcdef0

# Variable (no required args)
variable "region" {}
variable "full" {
  type        = string
  default     = "us-east-1"
  description = "AWS region"
}
```

---

*Generated from ExamTopics Terraform Associate v1, v003, v004 – optimized for memorization*
