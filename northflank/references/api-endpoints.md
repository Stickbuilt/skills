# Northflank API Endpoint Reference

Base URL: `https://api.northflank.com/v1/`

All requests use JSON. Auth via `Authorization: Bearer <token>` header.
Paginated endpoints accept `per_page` (max 100), `page`, and `cursor` query params.

## Table of Contents

1. [Projects](#projects)
2. [Services](#services)
3. [Jobs](#jobs)
4. [Addons](#addons)
5. [Secrets](#secrets)
6. [Global Secrets](#global-secrets)
7. [Domains & Subdomains](#domains--subdomains)
8. [Templates](#templates)
9. [Workflows](#workflows)
10. [Pipelines](#pipelines)
11. [Volumes](#volumes)
12. [Cloud / BYOC](#cloud--byoc)
13. [Log Sinks](#log-sinks)
14. [Notifications](#notifications)
15. [Registry Credentials](#registry-credentials)
16. [Load Balancers](#load-balancers)
17. [Backup Destinations](#backup-destinations)
18. [Tags](#tags)
19. [Billing](#billing)
20. [VCS](#vcs)
21. [Plans & Regions](#plans--regions)
22. [DNS](#dns)

---

## Projects

| Method | Path | Description |
|--------|------|-------------|
| GET | `/projects` | List projects (paginated) |
| POST | `/projects` | Create project |
| GET | `/projects/{projectId}` | Get project details |
| PUT | `/projects/{projectId}` | Replace project |
| PATCH | `/projects/{projectId}` | Partial update project |
| DELETE | `/projects/{projectId}` | Delete project |

**Project fields:** id, name, description, tags[], color, region

---

## Services

| Method | Path | Description |
|--------|------|-------------|
| GET | `/projects/{projectId}/services` | List services (paginated) |
| POST | `/projects/{projectId}/services/build` | Create build service |
| POST | `/projects/{projectId}/services/combined` | Create combined service |
| POST | `/projects/{projectId}/services/deployment` | Create deployment service |
| GET | `/projects/{projectId}/services/{serviceId}` | Get service details |
| PUT | `/projects/{projectId}/services/{serviceId}/build` | Replace build service |
| PUT | `/projects/{projectId}/services/{serviceId}/combined` | Replace combined service |
| PUT | `/projects/{projectId}/services/{serviceId}/deployment` | Replace deployment service |
| PATCH | `/projects/{projectId}/services/{serviceId}/build` | Patch build service |
| PATCH | `/projects/{projectId}/services/{serviceId}/combined` | Patch combined service |
| PATCH | `/projects/{projectId}/services/{serviceId}/deployment` | Patch deployment service |
| DELETE | `/projects/{projectId}/services/{serviceId}` | Delete service |

### Service Subresources

| Method | Path | Description |
|--------|------|-------------|
| GET | `.../services/{serviceId}/branches` | List branches (paginated) |
| GET | `.../services/{serviceId}/builds` | List builds (paginated) |
| POST | `.../services/{serviceId}/builds` | Start build |
| GET | `.../services/{serviceId}/builds/{buildId}` | Get build details |
| DELETE | `.../services/{serviceId}/builds/{buildId}` | Abort build |
| GET | `.../services/{serviceId}/build-arguments` | Get build arguments |
| POST | `.../services/{serviceId}/build-arguments` | Update build arguments |
| GET | `.../services/{serviceId}/build-argument-details` | Get build argument details |
| POST | `.../services/{serviceId}/build-options` | Update build options |
| POST | `.../services/{serviceId}/build-source` | Update build source |
| GET | `.../services/{serviceId}/containers` | List containers (paginated) |
| GET | `.../services/{serviceId}/deployment` | Get deployment details |
| POST | `.../services/{serviceId}/deployment` | Update deployment |
| GET | `.../services/{serviceId}/health-checks` | Get health checks |
| POST | `.../services/{serviceId}/health-checks` | Update health checks |
| POST | `.../services/{serviceId}/pause` | Pause service |
| GET | `.../services/{serviceId}/ports` | Get ports |
| POST | `.../services/{serviceId}/ports` | Update ports |
| GET | `.../services/{serviceId}/pull-requests` | List PRs (paginated) |
| POST | `.../services/{serviceId}/restart` | Restart service |
| POST | `.../services/{serviceId}/resume` | Resume service |
| GET | `.../services/{serviceId}/runtime-environment` | Get env vars |
| POST | `.../services/{serviceId}/runtime-environment` | Update env vars (REPLACES ALL) |
| GET | `.../services/{serviceId}/runtime-environment-details` | Get env var details |
| POST | `.../services/{serviceId}/scale` | Scale service |

### Service Response Shape
```json
{
  "id": "example-service",
  "appId": "/user/project/service",
  "projectId": "project-id",
  "name": "Example Service",
  "tags": [],
  "description": "",
  "serviceType": "combined",
  "disabledCI": false,
  "disabledCD": false,
  "status": {
    "build": { "status": "SUCCESS", "lastTransitionTime": "2021-11-29T11:47:16.624Z" },
    "deployment": { "status": "COMPLETED", "reason": "DEPLOYING", "lastTransitionTime": "..." }
  }
}
```

### Runtime Environment Response
```json
{
  "runtimeEnvironment": {
    "DATABASE_URL": "postgres://...",
    "API_KEY": "sk-..."
  }
}
```

---

## Jobs

| Method | Path | Description |
|--------|------|-------------|
| GET | `/projects/{projectId}/jobs` | List jobs (paginated) |
| POST | `/projects/{projectId}/jobs` | Create job |
| POST | `/projects/{projectId}/jobs/cron` | Create cron job |
| POST | `/projects/{projectId}/jobs/manual` | Create manual job |
| GET | `/projects/{projectId}/jobs/{jobId}` | Get job details |
| PUT | `/projects/{projectId}/jobs/{jobId}` | Replace job |
| PUT | `/projects/{projectId}/jobs/{jobId}/cron` | Replace cron job |
| PUT | `/projects/{projectId}/jobs/{jobId}/manual` | Replace manual job |
| PATCH | `/projects/{projectId}/jobs/{jobId}` | Patch job |
| PATCH | `/projects/{projectId}/jobs/{jobId}/cron` | Patch cron job |
| PATCH | `/projects/{projectId}/jobs/{jobId}/manual` | Patch manual job |
| DELETE | `/projects/{projectId}/jobs/{jobId}` | Delete job |

### Job Subresources

| Method | Path | Description |
|--------|------|-------------|
| GET | `.../jobs/{jobId}/branches` | List branches |
| GET | `.../jobs/{jobId}/builds` | List builds |
| POST | `.../jobs/{jobId}/builds` | Start build |
| GET | `.../jobs/{jobId}/builds/{buildId}` | Get build |
| DELETE | `.../jobs/{jobId}/builds/{buildId}` | Abort build |
| GET | `.../jobs/{jobId}/build-arguments` | Get build arguments |
| POST | `.../jobs/{jobId}/build-arguments` | Update build arguments |
| GET | `.../jobs/{jobId}/build-argument-details` | Get build argument details |
| POST | `.../jobs/{jobId}/build-options` | Update build options |
| POST | `.../jobs/{jobId}/build-source` | Update build source |
| GET | `.../jobs/{jobId}/containers` | List containers |
| GET | `.../jobs/{jobId}/deployment` | Get deployment |
| POST | `.../jobs/{jobId}/deployment` | Update deployment |
| GET | `.../jobs/{jobId}/health-checks` | Get health checks |
| POST | `.../jobs/{jobId}/health-checks` | Update health checks |
| POST | `.../jobs/{jobId}/pause` | Pause job |
| GET | `.../jobs/{jobId}/pull-requests` | List PRs |
| POST | `.../jobs/{jobId}/resume` | Resume job |
| GET | `.../jobs/{jobId}/runs` | List runs (paginated) |
| POST | `.../jobs/{jobId}/runs` | Start run |
| GET | `.../jobs/{jobId}/runs/{runId}` | Get run |
| DELETE | `.../jobs/{jobId}/runs/{runId}` | Abort run |
| GET | `.../jobs/{jobId}/runtime-environment` | Get env vars |
| POST | `.../jobs/{jobId}/runtime-environment` | Update env vars (REPLACES ALL) |
| GET | `.../jobs/{jobId}/runtime-environment-details` | Get env var details |
| POST | `.../jobs/{jobId}/scale` | Scale job |
| POST | `.../jobs/{jobId}/settings` | Update settings |
| POST | `.../jobs/{jobId}/suspend` | Suspend job |

---

## Addons

| Method | Path | Description |
|--------|------|-------------|
| GET | `/addon-types` | List addon types (no auth needed) |
| GET | `/projects/{projectId}/addons` | List addons (paginated) |
| POST | `/projects/{projectId}/addons` | Create addon |
| GET | `/projects/{projectId}/addons/{addonId}` | Get addon |
| PUT | `/projects/{projectId}/addons/{addonId}` | Replace addon |
| PATCH | `/projects/{projectId}/addons/{addonId}` | Patch addon |
| DELETE | `/projects/{projectId}/addons/{addonId}` | Delete addon |

### Addon Subresources

| Method | Path | Description |
|--------|------|-------------|
| GET | `.../addons/{addonId}/backup-schedules` | List backup schedules |
| POST | `.../addons/{addonId}/backup-schedules` | Create backup schedule |
| DELETE | `.../addons/{addonId}/backup-schedules/{scheduleId}` | Delete schedule |
| GET | `.../addons/{addonId}/backups` | List backups |
| POST | `.../addons/{addonId}/backups` | Create backup |
| GET | `.../addons/{addonId}/backups/{backupId}` | Get backup |
| DELETE | `.../addons/{addonId}/backups/{backupId}` | Delete backup |
| POST | `.../addons/{addonId}/backups/{backupId}/abort` | Abort backup |
| GET | `.../addons/{addonId}/backups/{backupId}/download` | Get download URL |
| POST | `.../addons/{addonId}/backups/{backupId}/restore` | Restore from backup |
| GET | `.../addons/{addonId}/backups/{backupId}/restores` | List restores |
| POST | `.../addons/{addonId}/backups/{backupId}/restores/{restoreId}/abort` | Abort restore |
| POST | `.../addons/{addonId}/backups/{backupId}/retain` | Retain backup |
| GET | `.../addons/{addonId}/containers` | List containers |
| GET | `.../addons/{addonId}/credentials` | Get credentials |
| POST | `.../addons/{addonId}/import-backup` | Import backup |
| POST | `.../addons/{addonId}/network-settings` | Update network settings |
| POST | `.../addons/{addonId}/pause` | Pause |
| POST | `.../addons/{addonId}/reset` | Reset |
| POST | `.../addons/{addonId}/restart` | Restart |
| GET | `.../addons/{addonId}/restores` | List restores |
| POST | `.../addons/{addonId}/resume` | Resume |
| POST | `.../addons/{addonId}/scale` | Scale |
| POST | `.../addons/{addonId}/security` | Update security |
| GET | `.../addons/{addonId}/version` | Get version |
| POST | `.../addons/{addonId}/version` | Update version |

### Addon Types Response Shape
```json
{
  "addonTypes": [{
    "type": "mongodb",
    "name": "MongoDB",
    "description": "...",
    "versions": ["6.0", "7.0"],
    "resources": { "storage": {...}, "replicas": {...} }
  }]
}
```

---

## Secrets

| Method | Path | Description |
|--------|------|-------------|
| GET | `/projects/{projectId}/secrets` | List secrets (paginated) |
| POST | `/projects/{projectId}/secrets` | Create secret |
| GET | `/projects/{projectId}/secrets/{secretId}` | Get secret |
| PUT | `/projects/{projectId}/secrets/{secretId}` | Replace secret |
| PATCH | `/projects/{projectId}/secrets/{secretId}` | Patch secret |
| DELETE | `/projects/{projectId}/secrets/{secretId}` | Delete secret |
| POST | `.../secrets/{secretId}/update` | Update secret data |
| POST | `.../secrets/{secretId}/link` | Update secret link |
| GET | `.../secrets/{secretId}/link` | Get secret link |
| DELETE | `.../secrets/{secretId}/link` | Delete secret link |
| GET | `.../secrets/{secretId}/details` | Get secret details (with values) |

---

## Global Secrets

| Method | Path | Description |
|--------|------|-------------|
| GET | `/secrets` | List global secrets (paginated) |
| POST | `/secrets` | Create global secret |
| GET | `/secrets/{secretId}` | Get global secret |
| PUT | `/secrets/{secretId}` | Replace |
| PATCH | `/secrets/{secretId}` | Patch |
| DELETE | `/secrets/{secretId}` | Delete |

---

## Domains & Subdomains

| Method | Path | Description |
|--------|------|-------------|
| GET | `/domains` | List domains (paginated) |
| POST | `/domains` | Create domain |
| GET | `/domains/{domainId}` | Get domain |
| DELETE | `/domains/{domainId}` | Delete domain |
| GET | `/domains/{domainId}/certificate` | Get certificate |
| POST | `/domains/{domainId}/certificate` | Import certificate |
| POST | `/domains/{domainId}/verify` | Verify domain |
| POST | `/domains/{domainId}/subdomains` | Add subdomain |
| GET | `.../subdomains/{subdomainId}` | Get subdomain |
| DELETE | `.../subdomains/{subdomainId}` | Delete subdomain |
| POST | `.../subdomains/{subdomainId}/assign` | Assign to service |
| DELETE | `.../subdomains/{subdomainId}/assign` | Unassign |
| POST | `.../subdomains/{subdomainId}/cdn/enable` | Enable CDN |
| POST | `.../subdomains/{subdomainId}/cdn/disable` | Disable CDN |
| POST | `.../subdomains/{subdomainId}/verify` | Verify subdomain |
| POST | `.../subdomains/{subdomainId}/paths` | Add path |
| GET | `.../subdomains/{subdomainId}/paths` | List paths |
| GET | `.../subdomains/{subdomainId}/paths/{pathId}` | Get path |
| DELETE | `.../subdomains/{subdomainId}/paths/{pathId}` | Delete path |
| POST | `.../subdomains/{subdomainId}/paths/{pathId}/update` | Update path |
| POST | `.../subdomains/{subdomainId}/paths/{pathId}/assign` | Assign path |
| DELETE | `.../subdomains/{subdomainId}/paths/{pathId}/assign` | Unassign path |

---

## Templates

| Method | Path | Description |
|--------|------|-------------|
| GET | `/templates` | List templates (paginated) |
| POST | `/templates` | Create template |
| GET | `/templates/{templateId}` | Get template |
| POST | `/templates/{templateId}` | Update template |
| DELETE | `/templates/{templateId}` | Delete template |
| POST | `/templates/{templateId}/run` | Run template |
| GET | `/templates/{templateId}/runs` | List runs (paginated) |
| GET | `/templates/{templateId}/runs/{runId}` | Get run |
| POST | `/templates/{templateId}/runs/{runId}/abort` | Abort run |

---

## Workflows

| Method | Path | Description |
|--------|------|-------------|
| GET | `/projects/{projectId}/workflows` | List workflows (paginated) |
| POST | `/projects/{projectId}/workflows` | Create workflow |
| GET | `.../workflows/{workflowId}` | Get workflow |
| POST | `.../workflows/{workflowId}` | Update workflow |
| POST | `.../workflows/{workflowId}/run` | Run workflow |
| GET | `.../workflows/{workflowId}/runs` | List runs (paginated) |
| GET | `.../workflows/{workflowId}/runs/{runId}` | Get run |
| POST | `.../workflows/{workflowId}/runs/{runId}/abort` | Abort run |

---

## Pipelines

| Method | Path | Description |
|--------|------|-------------|
| GET | `/projects/{projectId}/pipelines` | List pipelines (paginated) |
| GET | `.../pipelines/{pipelineId}` | Get pipeline |

Pipelines also have preview-templates, release-flows, and preview-blueprints — each with CRUD + run/abort patterns.

---

## Volumes

| Method | Path | Description |
|--------|------|-------------|
| GET | `/projects/{projectId}/volumes` | List volumes (paginated) |
| POST | `/projects/{projectId}/volumes` | Create volume |
| GET | `.../volumes/{volumeId}` | Get volume |
| POST | `.../volumes/{volumeId}` | Update volume |
| DELETE | `.../volumes/{volumeId}` | Delete volume |
| POST | `.../volumes/{volumeId}/attach` | Attach volume |
| POST | `.../volumes/{volumeId}/detach` | Detach volume |
| GET | `.../volumes/{volumeId}/backups` | List backups |
| POST | `.../volumes/{volumeId}/backups` | Create backup |
| GET | `.../volumes/{volumeId}/backups/{backupId}` | Get backup |
| DELETE | `.../volumes/{volumeId}/backups/{backupId}` | Delete backup |

---

## Cloud / BYOC

| Method | Path | Description |
|--------|------|-------------|
| GET | `/cloud-providers` | List providers |
| GET | `/cloud-providers/clusters` | List clusters (paginated) |
| POST | `/cloud-providers/clusters` | Create cluster |
| GET | `/cloud-providers/clusters/{clusterId}` | Get cluster |
| PUT | `/cloud-providers/clusters/{clusterId}` | Replace cluster |
| PATCH | `/cloud-providers/clusters/{clusterId}` | Patch cluster |
| DELETE | `/cloud-providers/clusters/{clusterId}` | Delete cluster |
| GET | `.../clusters/{clusterId}/nodes` | List nodes |
| POST | `.../clusters/{clusterId}/nodes/{nodeId}/cordon` | Cordon node |
| POST | `.../clusters/{clusterId}/nodes/{nodeId}/drain` | Drain node |
| POST | `.../clusters/{clusterId}/nodes/{nodeId}/uncordon` | Uncordon node |
| GET | `/cloud-providers/integrations` | List integrations |
| POST | `/cloud-providers/integrations` | Create integration |
| GET | `/cloud-providers/integrations/{integrationId}` | Get integration |
| PUT | `/cloud-providers/integrations/{integrationId}` | Replace |
| PATCH | `/cloud-providers/integrations/{integrationId}` | Patch |
| DELETE | `/cloud-providers/integrations/{integrationId}` | Delete |
| GET | `/cloud-providers/node-types` | List node types |
| GET | `/cloud-providers/regions` | List regions |

---

## Log Sinks

| Method | Path | Description |
|--------|------|-------------|
| GET | `/integrations/log-sinks` | List log sinks (paginated) |
| POST | `/integrations/log-sinks` | Create log sink |
| GET | `/integrations/log-sinks/{logSinkId}` | Get log sink |
| DELETE | `/integrations/log-sinks/{logSinkId}` | Delete |
| POST | `/integrations/log-sinks/{logSinkId}/pause` | Pause |
| POST | `/integrations/log-sinks/{logSinkId}/resume` | Resume |
| POST | `/integrations/log-sinks/{logSinkId}/update` | Update |

---

## Notifications

| Method | Path | Description |
|--------|------|-------------|
| GET | `/integrations/notifications` | List notifications |
| POST | `/integrations/notifications` | Create notification |
| GET | `/integrations/notifications/{notificationId}` | Get notification |
| POST | `/integrations/notifications/{notificationId}` | Update |
| DELETE | `/integrations/notifications/{notificationId}` | Delete |

---

## Registry Credentials

| Method | Path | Description |
|--------|------|-------------|
| GET | `/integrations/registries` | List registry credentials |
| POST | `/integrations/registries` | Add credentials |
| GET | `/integrations/registries/{credentialId}` | Get credentials |
| PATCH | `/integrations/registries/{credentialId}` | Update |
| DELETE | `/integrations/registries/{credentialId}` | Delete |

---

## Load Balancers

| Method | Path | Description |
|--------|------|-------------|
| GET | `/load-balancers` | List load balancers |
| POST | `/load-balancers` | Create |
| GET | `/load-balancers/{loadBalancerId}` | Get |
| PUT | `/load-balancers/{loadBalancerId}` | Replace |
| PATCH | `/load-balancers/{loadBalancerId}` | Patch |
| DELETE | `/load-balancers/{loadBalancerId}` | Delete |

---

## Backup Destinations

| Method | Path | Description |
|--------|------|-------------|
| GET | `/backup-destinations` | List destinations |
| POST | `/backup-destinations` | Add destination |
| GET | `/backup-destinations/{destinationId}` | Get (includes secrets) |
| PATCH | `/backup-destinations/{destinationId}` | Update |
| DELETE | `/backup-destinations/{destinationId}` | Delete |
| GET | `/backup-destinations/{destinationId}/backups` | List backups |

**Backup destination fields:** name, description, type ('s3'), prefix, credentials (accessKey, secretKey, bucketName, region, endpoint)

---

## Tags

| Method | Path | Description |
|--------|------|-------------|
| GET | `/tags` | List tags (paginated) |
| POST | `/tags` | Add tag |
| GET | `/tags/{tagId}` | Get tag |
| PUT | `/tags/{tagId}` | Replace tag |
| PATCH | `/tags/{tagId}` | Patch tag |
| DELETE | `/tags/{tagId}` | Delete tag |

---

## Billing

| Method | Path | Description |
|--------|------|-------------|
| GET | `/billing/invoices` | List invoices (paginated) |
| GET | `/billing/invoices/details?timestamp={ts}` | Get invoice details |

---

## VCS

| Method | Path | Description |
|--------|------|-------------|
| GET | `/integrations/vcs` | List VCS integrations |
| POST | `/integrations/vcs/custom/{vcsId}/token` | Create custom VCS token |
| GET | `/integrations/vcs/repos?vcsId={id}` | List repos |
| GET | `/integrations/vcs/repos/{repoId}/branches` | List branches |

---

## Plans & Regions

| Method | Path | Description |
|--------|------|-------------|
| GET | `/plans` | List available plans |
| GET | `/regions` | List available regions |

---

## DNS

| Method | Path | Description |
|--------|------|-------------|
| GET | `/dns-id` | Get DNS ID |
