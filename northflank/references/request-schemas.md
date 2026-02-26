# Northflank API Request Body Schemas

Detailed JSON body schemas for create/update operations. Read this when you need to construct payloads.

## Table of Contents

1. [Services](#services)
   - [Create Combined Service](#create-combined-service)
   - [Create Deployment Service](#create-deployment-service)
   - [Create Build Service](#create-build-service)
   - [Update Deployment](#update-service-deployment)
   - [Update Ports](#update-service-ports)
   - [Update Health Checks](#update-service-health-checks)
   - [Scale Service](#scale-service)
   - [Patch Services](#patch-services)
2. [Shared Sub-Schemas](#shared-sub-schemas)
   - [Deployment Block](#deployment-block)
   - [Ports](#ports)
   - [Health Checks](#health-checks)
   - [Autoscaling](#autoscaling)
   - [Load Balancing](#load-balancing)
   - [Port Security](#port-security)
   - [Build Settings](#build-settings)
   - [Build Configuration](#build-configuration)
3. [Addons](#addons)
4. [Jobs](#jobs)
5. [Projects](#projects)
6. [Domains & Subdomains](#domains--subdomains)
7. [Templates](#templates)
8. [Workflows](#workflows)
9. [Secrets](#secrets)
10. [Volumes](#volumes)
11. [Log Sinks](#log-sinks)

---

## Services

### Create Combined Service

`POST /projects/{projectId}/services/combined`

The most common service type — builds from source and deploys.

```jsonc
{
  "name": "My Service",               // required
  "description": "",                   // optional
  "tags": [],                          // optional
  "billing": {
    "deploymentPlan": "nf-compute-20", // required — plan ID
    "buildPlan": "nf-compute-200-8",   // optional — build plan
    "gpu": {                           // optional
      "enabled": true,
      "configuration": { "gpuType": "...", "gpuCount": 1, "timesliced": false }
    }
  },
  "infrastructure": { "architecture": "x86" },  // optional: "x86" | "arm"
  "vcsData": {                         // required for git-based builds
    "projectUrl": "https://github.com/user/repo",
    "projectType": "github",           // "bitbucket" | "gitlab" | "github" | "self-hosted" | "azure"
    "projectBranch": "main",           // required
    "selfHostedVcsId": "",             // required if projectType = "self-hosted"
    "accountLogin": "",                // optional — specific VCS account
    "vcsLinkId": ""                    // optional
  },
  "buildSettings": { /* see Build Settings */ },   // required
  "buildConfiguration": { /* see Build Configuration */ },  // optional
  "buildArguments": {},                // optional — key-value build args
  "buildFiles": {},                    // optional — secret files for build
  "deployment": { /* see Deployment Block */ },     // required
  "ports": [/* see Ports */],          // optional
  "runtimeEnvironment": { "KEY": "value" },  // optional — env vars
  "runtimeFiles": {},                  // optional — secret files at runtime
  "healthChecks": [/* see Health Checks */],  // optional
  "loadBalancing": { /* see Load Balancing */ },  // optional
  "autoscaling": { /* see Autoscaling */ },  // optional
  "disabledCI": false,                 // optional
  "createOptions": { "volumesToAttach": [] }  // optional
}
```

### Create Deployment Service

`POST /projects/{projectId}/services/deployment`

Deploys an external Docker image or internal build — no build step.

```jsonc
{
  "name": "My Service",               // required
  "description": "",
  "tags": [],
  "billing": {
    "deploymentPlan": "nf-compute-20"  // required
  },
  "deployment": {
    "instances": 1,                    // required
    // Pick ONE source:
    "external": {                      // Option A: external image
      "imagePath": "nginx:latest",     // required
      "credentials": ""               // optional — saved credentials ID
    },
    "internal": {                      // Option B: internal build service
      "id": "build-service-id",
      "branch": "main",
      "buildSHA": "latest"            // or specific SHA
    },
    // Common deployment fields:
    "docker": {
      "configType": "default"          // "default" | "customEntrypoint" | "customCommand" | "customEntrypointCustomCommand"
    },
    "storage": { "ephemeralStorage": { "storageSize": 1024 } },
    "strategy": { "type": "rollout-balanced" }
  },
  "ports": [/* see Ports */],
  "runtimeEnvironment": {},
  "healthChecks": [],
  "loadBalancing": {},
  "autoscaling": {}
}
```

### Create Build Service

`POST /projects/{projectId}/services/build`

Build-only service (no deployment). Used as a source for deployment services.

```jsonc
{
  "name": "My Build",
  "billing": {
    "buildPlan": "nf-compute-200-8"   // optional
  },
  "vcsData": {
    "projectUrl": "https://github.com/user/repo",
    "projectType": "github",
    "accountLogin": ""
  },
  "buildSettings": { /* see Build Settings */ },
  "buildConfiguration": { /* see Build Configuration */ },
  "buildArguments": {},
  "disabledCI": false
}
```

### Update Service Deployment

`POST /projects/{projectId}/services/{serviceId}/deployment`

Change what image/branch a service deploys.

```jsonc
// Variant 1: Deploy external image
{
  "external": {
    "imagePath": "myimage:v1.2.3",
    "credentials": ""
  },
  "docker": { "configType": "default" }
}

// Variant 2: Deploy internal build
{
  "internal": {
    "id": "build-service-id",
    "branch": "main",
    "buildSHA": "latest"
  }
}

// Variant 3: Just change runtime config
{
  "docker": {
    "configType": "customCommand",
    "customCommand": "./start.sh"
  }
}
```

### Update Service Ports

`POST /projects/{projectId}/services/{serviceId}/ports`

**REPLACES entire port list.** Pass `id` of existing ports to preserve security configs.

```jsonc
{
  "ports": [
    {
      "id": "existing-port-id",        // optional — preserves security config
      "name": "http",                   // required
      "internalPort": 8080,             // required
      "public": true,                   // optional
      "protocol": "HTTP",              // "HTTP" | "HTTP/2" | "TCP" | "UDP"
      "domains": [],                    // optional
      "disableNfDomain": false,         // optional
      "security": { /* see Port Security */ }
    }
  ]
}
```

### Update Service Health Checks

`POST /projects/{projectId}/services/{serviceId}/health-checks`

**REPLACES entire health check list.**

```jsonc
{
  "healthChecks": [/* see Health Checks schema */]
}
```

### Scale Service

`POST /projects/{projectId}/services/{serviceId}/scale`

```jsonc
{
  "instances": 2,                      // optional
  "deploymentPlan": "nf-compute-50",   // optional — change plan
  "storage": {
    "ephemeralStorage": { "storageSize": 2048 },
    "shmSize": 256
  },
  "gracePeriodSeconds": 30
}
```

### Patch Services

`PATCH /projects/{projectId}/services/{serviceId}/(build|combined|deployment)`

Same structure as create, but **all fields optional**. `name` is NOT patchable. When `deployment` object is provided in a patch, `instances` is still required.

---

## Shared Sub-Schemas

### Deployment Block

```jsonc
{
  "type": "deployment",               // "deployment" | "statefulSet"
  "instances": 1,                      // required
  "docker": {                          // for Docker images
    "configType": "default",           // "default" | "customEntrypoint" | "customCommand" | "customEntrypointCustomCommand"
    "customEntrypoint": "",
    "customCommand": ""
  },
  "buildpack": {                       // for buildpack builds
    "configType": "default",           // "default" | "customProcess" | "customCommand" | "customEntrypointCustomCommand" | "originalEntrypointCustomCommand"
    "customProcess": "",
    "customEntrypoint": "",
    "customCommand": ""
  },
  "storage": {
    "ephemeralStorage": { "storageSize": 1024 },  // MB
    "shmSize": 64                      // /dev/shm MB
  },
  "strategy": {
    "type": "rollout-balanced",        // "recreate" | "rollout-steady" | "rollout-balanced" | "rollout-fast" | "custom"
    "settings": { "maxSurge": 1, "maxUnavailable": 0 }  // only for "custom"
  },
  "zonalRedundancy": {
    "type": "disabled",                // "disabled" | "preferred" | "required"
    "minZones": 2                      // only if "required"
  },
  "gracePeriodSeconds": 30,            // SIGTERM to SIGKILL timeout
  "metadata": { "labels": {}, "annotations": {} }
}
```

### Ports

```jsonc
{
  "name": "http",                      // required — identifier
  "internalPort": 8080,                // required
  "public": true,                      // optional
  "protocol": "HTTP",                 // "HTTP" | "HTTP/2" | "TCP" | "UDP"
  "domains": ["sub.example.com"],      // optional
  "disableNfDomain": false,            // optional — disable *.code.run domain
  "advancedOptions": { "enableTlsPassthrough": false },
  "security": { /* see Port Security */ }
}
```

### Health Checks

```jsonc
{
  "protocol": "HTTP",                 // "HTTP" | "CMD" | "TCP"
  "type": "livenessProbe",            // "livenessProbe" | "readinessProbe" | "startupProbe"
  "path": "/health",                   // required for HTTP
  "port": 8080,                        // required for HTTP
  "cmd": "curl localhost",             // required for CMD
  "initialDelaySeconds": 10,           // required
  "periodSeconds": 30,                 // required
  "timeoutSeconds": 5,                 // required
  "failureThreshold": 3,              // required
  "successThreshold": 1               // optional
}
```

### Autoscaling

```jsonc
{
  "horizontal": {
    "enabled": true,
    "minReplicas": 1,
    "maxReplicas": 10,
    "cpu": { "enabled": true, "thresholdPercentage": 80 },
    "memory": { "enabled": false, "thresholdPercentage": 80 },
    "rps": { "enabled": false, "thresholdValue": 100 },
    "userMetrics": {
      "enabled": false,
      "exposedMetricsPath": "/metrics",
      "exposedMetricsPort": 9090,
      "metrics": [
        { "metricName": "queue_depth", "metricType": "gauge", "thresholdValue": 50 }
      ]
    }
  }
}
```

### Load Balancing

```jsonc
// Round robin (default)
{ "mode": "roundRobin" }

// Least connection
{ "mode": "leastConnection" }

// Consistent hash by IP
{ "mode": "consistentHash", "consistentHash": { "mode": "ip" } }

// Consistent hash by header
{ "mode": "consistentHash", "consistentHash": { "mode": "customHeader", "header": "X-User-Id" } }

// Consistent replica routing
{ "mode": "consistentReplicaRouting", "consistentReplicaRouting": { "mode": "path" } }
```

### Port Security

```jsonc
{
  "credentials": [                     // basic auth
    { "username": "admin", "password": "secret", "type": "basic-auth" }
  ],
  "ip": [                             // IP allow/deny
    { "addresses": ["1.2.3.4/32"], "action": "ALLOW" }
  ],
  "sso": {                            // SSO
    "organizationId": "org-id",
    "allowAnyOrgUsers": true,
    "directoryGroupIds": [],
    "validateInternalTraffic": false
  },
  "verificationMode": "or",           // "or" | "and" — how to combine security features
  "securePathConfiguration": {
    "enabled": true,
    "rules": [{
      "paths": [{ "path": "/api", "priority": 1 }],
      "accessMode": "protected",       // "protected" | "unprotected"
      "securityPolicies": { /* same credentials/ip/sso structure */ }
    }]
  }
}
```

### Build Settings

```jsonc
// Dockerfile build
{
  "dockerfile": {
    "buildEngine": "buildkit",         // "buildkit" | "kaniko"
    "dockerFilePath": "/Dockerfile",   // required
    "dockerWorkDir": "/",              // required
    "buildkit": {
      "useCache": true,                // persistent build cache
      "cacheStorageSize": 2048         // MB
    }
  },
  "storage": { "ephemeralStorage": { "storageSize": 4096 } }
}

// Buildpack build
{
  "buildpack": {
    "builder": "HEROKU_24",           // see builder list below
    "buildpackLocators": [],           // custom buildpacks
    "buildContext": "/",               // working directory
    "useCache": true
  }
}
```

**Buildpack builders:** HEROKU_24, HEROKU_22, HEROKU_22_CLASSIC, HEROKU_20, HEROKU_18, GOOGLE_22, GOOGLE_V1, CNB_ALPINE, CNB_BIONIC, PAKETO_JAMMY_TINY, PAKETO_JAMMY_BASE, PAKETO_JAMMY_FULL, PAKETO_TINY, PAKETO_BASE, PAKETO_FULL

### Build Configuration

```jsonc
{
  "pathIgnoreRules": ["docs/**"],      // .gitignore-style ignore rules
  "isAllowList": false,                // invert pathIgnoreRules
  "prRestrictions": [],                // PR build rules (build service only)
  "branchRestrictions": [],            // branch build rules (build service only)
  "ciIgnoreFlagsEnabled": false,       // enable commit ignore flags
  "ciIgnoreFlags": ["[skip ci]"],
  "dockerfileTarget": "production",    // multi-stage target
  "dockerCredentials": [],
  "includeGitFolder": false,
  "fullGitClone": false,
  "enableGitLfs": false
}
```

---

## Addons

`POST /projects/{projectId}/addons`

```jsonc
{
  "name": "My Database",              // required
  "description": "",
  "tags": [],
  "type": "postgresql",               // required — "postgresql" | "mongodb" | "redis" | "mysql" | etc.
  "version": "16-latest",             // required — "latest" or "16-latest" style
  "billing": {
    "deploymentPlan": "nf-compute-20", // required
    "storage": 6144,                   // required — MB
    "replicas": 1                      // required
  },
  "tlsEnabled": false,
  "externalAccessEnabled": false,
  "ipPolicies": [
    { "addresses": ["1.2.3.4/32"], "action": "ALLOW" }
  ],
  "typeSpecificSettings": {
    // PostgreSQL:
    "postgresqlWalLevel": "replica",                    // "replica" | "logical"
    "postgresqlConnectionPoolerEnabled": false,
    "postgresqlConnectionPoolerReplicas": 1,
    "postgresqlReadConnectionPoolerEnabled": false,
    "postgresqlReadConnectionPoolerReplicas": 1,
    // Redis:
    "redisMaxMemoryPolicy": "noeviction",               // "noeviction" | "allkeys-lru" | "allkeys-lfu" | "volatile-lru" | "volatile-lfu" | "allkeys-random" | "volatile-random" | "volatile-ttl"
    "redisSentinelEnabled": false,
    // MySQL:
    "mysqlHaModeEnabled": false,
    "mysqlRouterReplicas": 1
  },
  "customCredentials": { "dbName": "mydb" },            // optional
  "backupSchedules": [{
    "scheduling": {
      "interval": "daily",             // "hourly" | "daily" | "weekly"
      "minute": [0],                   // required
      "hour": [3],                     // required for daily/weekly
      "day": [0]                       // required for weekly (0=Mon, 6=Sun)
    },
    "backupType": "dump",              // "dump" | "snapshot"
    "retentionTime": 7,                // days (hourly max 7, daily max 60, weekly max 120)
    "compressionType": "gz"            // "gz" | "zstd" (dump only)
  }],
  "source": {                          // optional — fork from backup
    "projectId": "src-project",
    "addonId": "src-addon",
    "backupId": "backup-id"
  },
  "metadata": { "labels": {}, "annotations": {} }
}
```

---

## Jobs

`POST /projects/{projectId}/jobs`

```jsonc
{
  "name": "My Job",                    // required
  "description": "",
  "tags": [],
  "billing": {
    "deploymentPlan": "nf-compute-20", // required
    "buildPlan": "nf-compute-200-8"    // optional
  },
  "deployment": {
    // Pick ONE source (same as service deployment):
    "vcs": {                           // Option A: build from git
      "projectUrl": "https://github.com/user/repo",
      "projectType": "github",
      "projectBranch": "main"
    },
    "external": {                      // Option B: external image
      "imagePath": "myimage:latest"
    },
    "internal": {                      // Option C: internal build
      "id": "build-service-id",
      "buildSHA": "latest"
    },
    // Common:
    "docker": { "configType": "default" },
    "storage": { "ephemeralStorage": { "storageSize": 1024 } },
    "gracePeriodSeconds": 30
  },
  "buildSettings": { /* see Build Settings — only for VCS */ },
  "runtimeEnvironment": { "KEY": "value" },
  "healthChecks": [],
  "settings": {
    "backoffLimit": 3,                 // required — retry attempts
    "runOnSourceChange": "never",      // "never" | "cd-promote" | "always"
    "activeDeadlineSeconds": 3600,     // max runtime
    "cron": {                          // presence makes it a cron job
      "schedule": "30 8 * * *",        // cron expression
      "suspended": false,
      "concurrencyPolicy": "forbid"    // "allow" | "forbid" | "replace"
    }
  }
}
```

**CreateJobCronEndpoint** and **CreateJobManualEndpoint** are identical but with `backoffLimit`, `runOnSourceChange`, `activeDeadlineSeconds`, `schedule`, `suspended`, `concurrencyPolicy` as top-level fields instead of nested under `settings`.

---

## Projects

`POST /projects`

```jsonc
{
  "name": "My Project",               // required
  "description": "",
  "color": "#EF233C",                 // hex color
  "region": "europe-west",            // OR "clusterId" for BYOC
  "networking": {                      // optional
    "allowedIngressProjects": [],
    "hostAliases": {
      "enabled": false,
      "hostEntries": [
        { "ipAddress": "10.0.0.1", "hostnames": ["db.internal"] }
      ]
    }
  }
}
```

---

## Domains & Subdomains

### Create Domain

`POST /domains`

```jsonc
{
  "domain": "example.com"             // required
}
```

### Add Subdomain

`POST /domains/{domain}/subdomains`

```jsonc
{
  "subdomain": "api",                 // required — creates api.example.com
  "options": {
    "tlsMode": "default",             // "default" | "passthrough"
    "minTlsProtocolVersion": "TLSV1_2"  // "TLSV1_2" | "TLSV1_3"
  }
}
```

### Assign Subdomain to Service

`POST /domains/{domain}/subdomains/{subdomain}/assign`

```jsonc
{
  "serviceId": "my-service",          // required
  "projectId": "my-project",          // required
  "portName": "http"                   // required — must match a port name
}
```

---

## Templates

`POST /templates`

```jsonc
{
  "name": "Deploy Template",          // required
  "description": "",
  "apiVersion": "v1.2",               // required
  "arguments": {                       // optional — referenced as ${args.argName}
    "IMAGE_TAG": { "default": "latest" }
  },
  "spec": { /* template specification */ },  // required (or use gitops)
  "gitops": {                          // optional — source from git
    "vcsService": "github",
    "repoUrl": "https://github.com/user/repo",
    "branch": "main",
    "filePath": "/template.json"
  },
  "options": {
    "autorun": false,
    "concurrencyPolicy": "allow",      // "allow" | "queue" | "forbid"
    "runOnCreation": false
  }
}
```

---

## Workflows

`POST /projects/{projectId}/workflows`

```jsonc
{
  "name": "My Workflow",              // required
  "description": "",
  "apiVersion": "v1.2",               // required
  "arguments": {},
  "spec": { /* workflow specification */ },  // required
  "options": {
    "autorun": false,
    "concurrencyPolicy": "allow"
  }
}
```

---

## Secrets

`POST /projects/{projectId}/secrets`

```jsonc
{
  "name": "App Secrets",              // required
  "description": "",
  "tags": [],
  "secretType": "environment",         // required: "environment-arguments" | "environment" | "arguments"
  "type": "secret",                    // "secret" | "config"
  "priority": 10,                      // required — merge priority
  "restrictions": {
    "restricted": false,
    "nfObjects": [
      { "id": "service-id", "type": "service" }
    ],
    "tags": [],
    "tagMatchCondition": "and"
  },
  "addonDependencies": [{             // link addon credentials
    "addonId": "my-postgres",
    "keys": [
      { "keyName": "HOST", "aliases": ["DB_HOST"] },
      { "keyName": "PORT", "aliases": ["DB_PORT"] }
    ]
  }],
  "secrets": {
    "variables": { "API_KEY": "sk-...", "NODE_ENV": "production" },
    "files": { "/etc/config.json": { "data": "base64...", "encoding": "utf-8" } }
  }
}
```

---

## Volumes

### Create Volume

`POST /projects/{projectId}/volumes`

```jsonc
{
  "name": "data-volume",              // required
  "tags": [],
  "mounts": [{                         // required
    "containerMountPath": "/data",     // required — path in container
    "volumeMountPath": ""              // optional — path within volume
  }],
  "spec": {
    "accessMode": "ReadWriteOnce",     // "ReadWriteOnce" | "ReadWriteMany"
    "storageSize": 10240               // required — MB
  },
  "attachedObjects": [{               // optional — auto-attach
    "id": "my-service",
    "type": "service"                  // "service" | "job"
  }]
}
```

### Attach Volume

`POST /projects/{projectId}/volumes/{volumeId}/attach`

```jsonc
{
  "nfObject": {
    "id": "my-service",
    "type": "service"
  }
}
```

---

## Log Sinks

`POST /integrations/log-sinks`

All log sinks share common fields plus type-specific `sinkData`:

```jsonc
{
  "name": "My Logs",                   // required
  "description": "",
  "restricted": false,                 // only send from listed projects
  "projects": ["project-id"],          // if restricted
  "sinkType": "...",                   // required — see variants below
  "sinkData": { /* type-specific */ },
  "options": {
    "useCustomLabels": false,          // parse JSON log lines for labels
    "forwardCdnLogs": false,
    "forwardIngressLogs": false,
    "forwardMeshLogs": false
  }
}
```

### Sink Types

**Loki:**
```jsonc
{ "sinkType": "loki", "sinkData": { "endpoint": "https://loki.example.com", "auth": { "strategy": "basic", "user": "...", "password": "..." } } }
```

**Datadog:**
```jsonc
{ "sinkType": "datadog_logs", "sinkData": { "default_api_key": "...", "region": "us" } }
// regions: "eu" | "us" | "us3" | "us5"
```

**Papertrail:**
```jsonc
{ "sinkType": "papertrail", "sinkData": { "authenticationStrategy": "port", "host": "logs.papertrail.com", "port": 12345 } }
// or: { "authenticationStrategy": "token", "uri": "...", "token": "..." }
```

**HTTP (generic):**
```jsonc
{ "sinkType": "http", "sinkData": { "uri": "https://logs.example.com", "encoding": { "codec": "json" }, "auth": { "strategy": "bearer", "token": "..." } } }
// auth strategies: "none" | "basic" (user/password) | "bearer" (token)
```

**AWS S3:**
```jsonc
{ "sinkType": "aws_s3", "sinkData": { "endpoint": "...", "region": "us-east-1", "bucket": "my-logs", "compression": "gzip", "auth": { "accessKeyId": "...", "secretAccessKey": "..." } } }
```

**Axiom:**
```jsonc
{ "sinkType": "axiom", "sinkData": { "dataset": "logs", "token": "...", "tokenType": "api" } }
// tokenType: "personal" (needs orgId) | "api"
```

**Better Stack:**
```jsonc
{ "sinkType": "betterStack", "sinkData": { "token": "...", "uri": "https://in.logs.betterstack.com" } }
```

**Honeycomb:**
```jsonc
{ "sinkType": "honeycomb", "sinkData": { "api_key": "...", "dataset": "my-dataset" } }
```

**LogDNA:**
```jsonc
{ "sinkType": "logdna", "sinkData": { "api_key": "..." } }
```

**Logz.io:**
```jsonc
{ "sinkType": "logzio", "sinkData": { "region": "us", "token": "..." } }
// regions: "eu" | "uk" | "us" | "ca" | "au" | "nl" | "wa"
```

**New Relic:**
```jsonc
{ "sinkType": "newRelic", "sinkData": { "accountId": "...", "licenseKey": "...", "region": "us" } }
```

**SolarWinds:**
```jsonc
{ "sinkType": "solarWinds", "sinkData": { "api_key": "...", "endpointType": "bulk" } }
```
