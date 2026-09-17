# Kapten concept model

Read this reference when identifying a Kapten resource, explaining how resources relate, or planning a change across the hierarchy.

## Resource hierarchy

```text
Organization
├── Clusters
├── Integrations and organization settings
└── Projects
    └── Environments ── targets one Cluster
        ├── Components
        │   ├── Applications
        │   ├── Jobs
        │   └── Databases
        └── Pipelines
            └── ordered Stages
                └── references to Components
```

Most CLI commands below the Organization require parent IDs as well as the target resource ID. Preserve the hierarchy shown by command help instead of treating IDs as globally interchangeable.

## Organization

An **Organization** is the tenant, membership, and top-level authorization boundary. Projects, Clusters, and Integrations belong to one Organization. OAuth login binds a CLI context to the Organization that was active during login; the Engine derives tenant authority from that authenticated context.

A local **CLI context** is not a platform resource. It records an identity, Organization, issuer, API endpoint, and credential reference so one user can switch safely between Organizations or installations.

## Project

A **Project** is an Organization-owned container for related Environments. Use it to group one product or workload across deployment boundaries. An Environment belongs to exactly one Project.

Project scope is significant for authorization and navigation. A Project can be deleted only when it has no child Environments; deleting it is not a cascade operation.

## Cluster

A **Cluster** is an Organization-owned compute target registered with Kapten. Environments reference a Cluster; workloads reach a Cluster through their Environment rather than owning cluster placement independently.

Cluster access is separate from Project and Environment access. Seeing a Project does not imply permission to inspect every Cluster in the Organization.

## Environment

An **Environment** is the deployment boundary inside a Project. It targets one Cluster, has a type such as development, staging, or production, and owns Components and Pipelines.

Environment authorization governs its contents. Components are not independent authorization boundaries. An Environment can be deleted only after its contained resources are removed.

## Component

A **Component** is the common inventory and lifecycle abstraction for a deployable workload in an Environment. The API may expose component kinds such as `KApplication`, `Kjob`, `KDatabase`, or `KHelm`; user-facing commands normally use resource-specific terms.

A component's configuration and current resource state are distinct from its run history:

- **Application**: a long-running workload built from a container image or source repository, with runtime resources, ports, health checks, storage, and scaling settings.
- **Job**: a finite container workload, either one-time or scheduled, with execution limits and restart behavior.
- **Database**: a PostgreSQL, MySQL, MongoDB, or Redis service. Provisioning can be in-cluster or managed, depending on engine and configuration.

## Pipeline

A **Pipeline** is an Environment-owned orchestration definition. It contains ordered Stages; each Stage selects Components and defines failure behavior. A Pipeline Run is the aggregate execution record, while its component actions produce Component Runs.

Do not confuse a Pipeline definition with a Pipeline Run. Editing stages changes future orchestration; inspecting a run describes one past or current execution.

## Runs and logs

A **Component Run** is one attempted component action, including an action initiated by a Pipeline. It carries status, timing, trigger, and failure information. The component's `last_run_id` and `last_run_status` point to the latest attempted action, not necessarily the latest successful one.

A **Pipeline Run** aggregates the execution of one Pipeline and its Stage runs. Run-scoped logs and timeline information should be queried with the run ID so events are attributed to the correct attempt.

## Authorization implications

Kapten evaluates permissions server-side against the active Organization and resource hierarchy. Membership alone does not imply every operation. Parent visibility can be derived from access to a child without granting sibling or parent mutation rights. Treat a successful list as the caller's authorized view, not a complete Organization inventory.
