---
name: northflank
description: >
  Comprehensive reference for the Northflank CLI and API — managing services, jobs, addons, projects,
  env vars, deployments, domains, secrets, templates, and all other Northflank resources. Use this skill
  PROACTIVELY whenever the user mentions Northflank, deploying to Northflank, the `northflank` CLI,
  Northflank API calls, or managing any Northflank resources (services, addons, projects, env vars,
  domains, templates, etc.). Also use when the user is working with Northflank even if they don't say
  "Northflank" explicitly — e.g., if they reference a Northflank project ID, service ID, or use
  patterns like `--projectId` or talk about runtime environments on their deployment platform.
---

# Northflank CLI & API Reference

## CLI Basics

Binary: `northflank`. Commands are `<verb> <resource> [subresource]` with kebab-case naming.

### Command Structure
```
northflank <verb> <resource> [subresource] [options]
```

**Verbs:** list, get, create, update, patch, put, delete, add, import, pause, resume, scale, start, suspend, restart, reset, run, abort, backup, restore, retain, assign, unassign, verify, enable, disable, attach, detach, cordon, drain, uncordon

**Special commands:** forward|fwd, command-exec|exec, ssh, download|dl, upload|ul, login, context|contexts, command-overview

### Common Options (all commands)
```
--verbose              Verbose output
--quiet                No console output
--skipValidation       Skip client-side validation
--noDefaults           Don't use context defaults
-o --output <OUTPUT>   json | yaml | custom-columns=col1,col2,...
```

### Resource Identifiers
```
--project  / --projectId <ID>
--service  / --serviceId <ID>
--job      / --jobId <ID>
--addon    / --addonId <ID>
```

### Mutation Input (create/update/patch/put)
```
-i, --input <JSON>         Inline JSON (takes precedence)
-f, --file <path>          Load from JSON/YAML file
```

### Pagination (list commands)
```
--loadAll          Load all pages
--per_page <N>     Results per page (max 100)
--page <N>         Page number
--cursor <STR>     Cursor for next page
```

### Resource Aliases
services=svc, projects=prj, addons=adn, domains=dmn, secrets=scrt, volumes=vol, credentials=cred, templates=tpl, invoices=inv, regions=rgn

### Context System
```bash
northflank login [--token-login] [--name <NAME>] [--token <TOKEN>] [--host <HOST>]
northflank context ls                    # list contexts
northflank context show                  # show current
northflank context use                   # switch context
northflank context remove|delete         # remove context
northflank context update-token|set-token
```

If you don't pass required path parameters (like `--projectId`), the CLI will interactively prompt. The `--noDefaults` flag forces prompting even when context defaults are set.

---

## Most Common Operations

### Services (service|services|svc)

```bash
# List services in a project
northflank list services --project <ID>

# Get service details
northflank get service --project <ID> --service <ID>

# Get subresources
northflank get service <subresource> --project <ID> --service <ID>
# subresources: branches, builds, build-arguments, build-argument-details, build,
#   containers, deployment, health-checks, ports, pull-requests,
#   runtime-environment, runtime-environment-details, logs, metrics,
#   build-logs, build-metrics

# Create service (types: build, combined, deployment)
northflank create service combined --project <ID> -i '<JSON>'

# Update subresources
northflank update service <subresource> --project <ID> --service <ID> -i '<JSON>'
# subresources: build-arguments, build-options, build-source, deployment,
#   health-checks, ports, runtime-environment

# Lifecycle
northflank pause service --project <ID> --service <ID>
northflank resume service --project <ID> --service <ID>
northflank restart service --project <ID> --service <ID>
northflank scale service --project <ID> --service <ID> -i '{"instances": 2}'
northflank start service build --project <ID> --service <ID>
northflank abort service build --project <ID> --service <ID> --buildId <ID>
northflank delete service --project <ID> --service <ID>
```

### Runtime Environment (Env Vars) — CRITICAL GOTCHA

The runtime environment update **REPLACES** all env vars. Always GET first, then include ALL existing vars plus new ones.

```bash
# Step 1: Get current env vars
northflank get service runtime-environment --project <ID> --service <ID> -o json
# Returns: { "runtimeEnvironment": { "KEY": "value", ... } }

# Step 2: Update with ALL vars (old + new)
northflank update service runtime-environment --project <ID> --service <ID> -i '{
  "runtimeEnvironment": {
    "EXISTING_VAR_1": "keep-this",
    "EXISTING_VAR_2": "keep-this-too",
    "NEW_VAR": "new-value"
  }
}'
```

This same pattern applies to job runtime environments — replace `service` with `job` and `--service` with `--job`.

### Jobs (job|jobs)

```bash
northflank list jobs --project <ID>
northflank get job --project <ID> --job <ID>

# Subresources: branches, builds, build-arguments, build-argument-details, build,
#   containers, deployment, health-checks, pull-requests, runs, run,
#   runtime-environment, runtime-environment-details, logs, metrics,
#   build-logs, build-metrics

northflank create job cron|manual --project <ID> -i '<JSON>'
northflank start job build|run --project <ID> --job <ID>
northflank abort job build|run --project <ID> --job <ID>
northflank pause|resume|scale|suspend job --project <ID> --job <ID>
northflank update job settings --project <ID> --job <ID> -i '<JSON>'
```

### Addons (addon|addons|adn)

```bash
northflank list addons --project <ID>
northflank get addon --project <ID> --addon <ID>
northflank get addon types                          # no project needed
northflank get addon credentials --project <ID> --addon <ID> -o json

# Subresources: backup-schedules, backups, backup, containers, credentials,
#   restores, version, logs, metrics
# Backup subresources: download, restores, logs

northflank create addon --project <ID> -i '<JSON>'
northflank pause|resume|restart|reset|scale addon --project <ID> --addon <ID>
northflank backup addon --project <ID> --addon <ID>
northflank restore addon backup --project <ID> --addon <ID> --backupId <ID>

# Update addon subresources
northflank update addon network-settings --project <ID> --addon <ID> -i '<JSON>'
northflank update addon security --project <ID> --addon <ID> -i '<JSON>'
northflank update addon version --project <ID> --addon <ID> -i '<JSON>'
```

### External Addons (third-party cloud resources via OpenTofu)

```bash
northflank list external-addons --project <ID>
northflank get external-addon --project <ID> --addon <ID>
northflank create external-addon --project <ID> -i '<JSON>'
northflank update external-addon --project <ID> --addon <ID> -i '<JSON>'
northflank delete external-addon --project <ID> --addon <ID>
```

### Projects (project|projects|prj)

```bash
northflank list projects
northflank get project --project <ID>
northflank create project -i '{"name": "My Project", "id": "my-project"}'
northflank delete project --project <ID>
```

### Secrets (secret|secrets|scrt)

```bash
northflank list secrets --project <ID>
northflank get secret --project <ID> --secret <ID>
northflank get secret-details --project <ID> --secret <ID>   # includes values
northflank create secret --project <ID> -i '<JSON>'
northflank update secret --project <ID> --secret <ID> -i '<JSON>'
```

### Global Secrets

```bash
northflank list global-secrets
northflank get global-secret --globalSecret <ID>
northflank create global-secret -i '<JSON>'
```

### Domains & Subdomains (domain|domains|dmn)

```bash
northflank list domains
northflank create domain -i '<JSON>'
northflank add domain subdomain --domain <ID> -i '<JSON>'
northflank assign subdomain service --domain <ID> --subdomain <ID> -i '<JSON>'
northflank verify domain|subdomain --domain <ID>
northflank enable|disable subdomain cdn --domain <ID> --subdomain <ID>
```

### Templates (template|templates|tpl)

```bash
northflank list templates
northflank get template --template <ID>
northflank run template --template <ID> -i '{"arguments": {"key": "value"}}'
northflank list template-runs --template <ID>
northflank get template-run --template <ID> --templateRun <ID>
northflank abort template-run --template <ID> --templateRun <ID>
```

### Volumes (volume|volumes|vol)

```bash
northflank list volumes --project <ID>
northflank create volume --project <ID> -i '<JSON>'
northflank attach volume --project <ID> --volume <ID> -i '<JSON>'
northflank detach volume --project <ID> --volume <ID>
northflank backup volume --project <ID> --volume <ID>
```

### Other Resources

```bash
# Workflows
northflank list workflows --project <ID>
northflank run workflow --project <ID> --workflow <ID>

# Log sinks
northflank list log-sinks
northflank create log-sink -i '<JSON>'
northflank pause|resume log-sink --logSink <ID>

# Registry credentials
northflank list registry-credentials
northflank add registry-credentials -i '<JSON>'

# SSH identities
northflank list ssh-identities
northflank add ssh-identities -i '<JSON>'
northflank get ssh-identities --sshIdentity <ID>
northflank update ssh-identities --sshIdentity <ID> -i '<JSON>'
northflank delete ssh-identities --sshIdentity <ID>

# Load balancers
northflank list load-balancers
northflank create load-balancer -i '<JSON>'

# Egress IPs
northflank list egress-ips
northflank create egress-ip -i '<JSON>'
northflank get egress-ip --egressIp <ID>
northflank delete egress-ip --egressIp <ID>

# Gradual rollout strategies
northflank create gradual-rollout-strategy -i '<JSON>'
northflank delete gradual-rollout-strategy --strategyId <ID>

# LLM model deployments
northflank list llm-model-deployments --project <ID>
northflank get llm-model-deployment --project <ID> --llmModelDeployment <ID>
northflank create llm-model-deployment --project <ID> -i '<JSON>'
northflank update llm-model-deployment --project <ID> --llmModelDeployment <ID> -i '<JSON>'
northflank delete llm-model-deployment --project <ID> --llmModelDeployment <ID>

# Tags
northflank list tags
northflank add tag -i '<JSON>'

# Billing
northflank list invoices
northflank get invoice details

# Cloud/BYOC
northflank list cloud providers|clusters|integrations|node-types|regions
northflank list cloud cluster nodes --cluster <ID>
northflank create cloud cluster|integration -i '<JSON>'
northflank patch cloud cluster --cluster <ID> -i '<JSON>'
northflank cordon|drain|uncordon cloud cluster node --cluster <ID> --node <ID>

# SSH into services
northflank ssh service --project <ID> --service <ID>

# VCS
northflank list vcs
northflank list repos --vcs <ID>
northflank list branches --vcs <ID> --repo <REPO>

# Plans & Regions
northflank list plans
northflank list regions

# Org roles & teams
northflank list org-roles
northflank create org-role -i '<JSON>'
northflank create org-role-member -i '<JSON>'
northflank list teams
northflank create team -i '<JSON>'
northflank list team-members --team <ID>
northflank list team-roles --team <ID>
northflank create team-role --team <ID> -i '<JSON>'

# Release flows
northflank get release-flow --project <ID> --releaseFlow <ID>
northflank update release-flow --project <ID> --releaseFlow <ID> -i '<JSON>'
northflank run release-flow --project <ID> --releaseFlow <ID> -i '<JSON>'
northflank list release-flow-runs --project <ID> --releaseFlow <ID>
northflank abort release-flow-run --project <ID> --releaseFlow <ID> --releaseFlowRun <ID>

# Preview blueprints
northflank list preview-blueprints --project <ID>
northflank create preview-blueprint --project <ID> -i '<JSON>'
northflank run preview-blueprint --project <ID> --previewBlueprint <ID>
northflank pause|resume blueprint-template-preview --project <ID> --preview <ID>

# Preview templates
northflank get preview-template --project <ID> --previewTemplate <ID>
northflank update preview-template --project <ID> --previewTemplate <ID> -i '<JSON>'
northflank run preview-template --project <ID> --previewTemplate <ID>

# Subdomain paths
northflank add subdomain path --domain <ID> --subdomain <ID> -i '<JSON>'
northflank list subdomain path --domain <ID> --subdomain <ID>
northflank assign subdomain path --domain <ID> --subdomain <ID> --path <ID> -i '<JSON>'
northflank unassign subdomain path --domain <ID> --subdomain <ID> --path <ID>
northflank delete subdomain path --domain <ID> --subdomain <ID> --path <ID>
```

---

## API Reference

Base URL: `https://api.northflank.com/v1/`
Auth: `Authorization: Bearer <token>`
Content-Type: `application/json`

**Reference files** (read as needed):
- `references/api-endpoints.md` — Complete endpoint table with all paths, methods, and response shapes
- `references/request-schemas.md` — Full JSON body schemas for create/update operations (services, addons, jobs, projects, domains, templates, secrets, volumes, log sinks)

### Key Types

**Service types:** `combined` | `build` | `deployment`

**Build status:** QUEUED, PENDING, UNSCHEDULABLE, STARTING, CLONING, BUILDING, UPLOADING, ABORTED, FAILURE, SUBMISSION_FAILURE, SUCCESS, CRASHED, IN_PROGRESS

**Deployment status:** PENDING, IN_PROGRESS, COMPLETED, FAILED (reason: SCALING | DEPLOYING)

**Addon status:** preDeployment, triggerAllocation, allocating, postDeployment, running, paused, scaling, upgrading, resetting, backup, restore, failed, error, errorAllocating, deleting, deleted

**Template run status:** pending, running, success, failure, aborted, aborting, queued, unknown, skipped, waiting, retrying, async_wait, approval_wait

**Metrics:** cpu, memory, networkIngress, networkEgress, tcpConnectionsOpen, diskUsage, requests, http4xxResponses, http5xxResponses, bandwidth, bandwidthVolume

---

## Common Recipes

### Create a combined service with Docker image
```bash
northflank create service combined --project my-project -i '{
  "name": "My Service",
  "id": "my-service",
  "deployment": {
    "instances": 1,
    "docker": { "image": "my-image:latest" }
  },
  "ports": [{ "name": "http", "internalPort": 8080, "public": true, "protocol": "HTTP" }]
}'
```

### Get addon connection credentials
```bash
northflank get addon credentials --project my-project --addon my-addon -o json
```

### Trigger a template run with arguments
```bash
northflank run template --template my-template -i '{
  "arguments": { "IMAGE_TAG": "v1.2.3" }
}'
```

### Check service build status
```bash
northflank get service builds --project my-project --service my-service -o json
```
