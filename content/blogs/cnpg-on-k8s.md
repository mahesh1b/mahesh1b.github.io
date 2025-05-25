---
date: '2025-05-24T11:39:45+05:30'
draft: false
title: 'Deploying PostgreSQL with CloudNativePG (CNPG) on Kubernetes'
tags: ["postgresql", "k8s", "cnpg", "cloudnativepg"]
cover: 
    image: images/cnpg-on-k8s.jpg
    responsiveImages: true
    linkFullImages: true
---

Kubernetes has long been the go-to platform for running stateless microservices—thanks to its high availability, scalability, and self-healing capabilities. However, stateful applications like databases were often avoided. Why? Managing stateful workloads introduces challenges like data consistency, backups, and disaster recovery. But now, [CloudNativePG (CNPG)](https://cloudnative-pg.io/) is changing the game, making PostgreSQL on Kubernetes not just possible but production-ready.

### What is CloudNativePG

CloudNativePG (CNPG) is an open-source Kubernetes operator designed to manage the lifecycle of PostgreSQL databases in a Kubernetes cluster. In short, PostgreSQL workloads can be managed using Kubernetes Custom Resource Definitions (CRDs).

### Install CloudNativePG

There are multiple ways to install the CNPG operator, including Kubernetes manifests, Helm charts, and the CNPG plugin. Here, we’ll use Helm charts with Terraform to deploy the operator.
```tf
resource "helm_release" "cnpg" {
  name = "cnpg"

  repository       = "https://cloudnative-pg.github.io/charts"
  chart            = "cloudnative-pg"
  namespace        = "cnpg-system"
  version          = "0.23.0"
  create_namespace = true
}
```
This deploys the CNPG Helm chart (version 0.23.0) in the `cnpg-system` namespace.

### Deploy a PostgreSQL cluster

To deploy a PostgreSQL cluster with High Availability (HA)—consisting of one primary and two replicas—use the following YAML configuration:
```yaml
---
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: test-db
  namespace: postgres
spec:
  instances: 3
  bootstrap:
    initdb:
      database: test
      owner: test
      secret:
        name: app-secret

  storage:
    size: 2Gi
---
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
  namespace: postgres
data:
  username: dGVzdA== # base64 encoded string
  password: cGFzc3dvcmQ= # base64 encoded string
```
Here, we create a `test-db` cluster using the `Cluster` CRD in the `postgres` namespace. The `bootstrap` section allows you to either create a new database from scratch (with `initdb`) or initialize a new cluster from an existing PostgreSQL cluster (using `pg_basebackup` or `recovery`).

By default, the CNPG operator creates:
- A default database named app.
- A superuser and an application user (same name as the database).
- A randomly generated password (unless specified otherwise).

You can also provide your own credentials by storing them in a Kubernetes secret and referencing it in the configuration.

> Note: The Kubernetes secret must be of type kubernetes.io/basic-auth, where the keys are username and password.

### Monitoring with Prometheus

The CNPG operator provides a Prometheus metrics exporter for each PostgreSQL instance, running on port `9187`. To monitor the cluster, you can use the Prometheus Operator.

Prerequisites:
- Ensure the Prometheus Operator is installed on your cluster.
- Update the CNPG Helm chart with the following values:
```yaml
monitoring:
  podMonitorEnabled: true
  # This allows prometheus-operator to scrape metrics from podMonitor.
  podMonitorAdditionalLabels:
    release: prometheus

  # This creates a CNPG Dashboard configmap that can be used in Grafana.
  grafanaDashboard:
    create: true
```
The PostgreSQL exporter exposes two types of metrics:
  - Predefined metrics (prefixed with `cnpg_collector_*`).
  - User-defined metrics (custom queries, currently in beta).

To define custom metrics, create a Kubernetes ConfigMap or Secret and reference it in the `Cluster` resource under `spec.monitoring.customQueriesConfigMap` or `customQueriesSecret`.
```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: test-db
  namespace: postgres
spec:

  # rest of the config

  monitoring:
    customQueriesConfigMap:
      - name: example-monitoring
        key: custom-queries
```

### Backups

CNPG supports two main methods for backing up PostgreSQL clusters:
1. Via CNPG-I plugins
2. Natively:
    - Object storage via Barman Cloud
    - Kubernetes Volume Snapshots

#### Backing Up to Object Storage (AWS S3 Example)
To enable backups, configure the `spec.backup` section in the Cluster CRD:
```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: test-db
  namespace: postgres
spec:
  
  # rest of the config

  serviceAccountTemplate:
    # For backup and restore, we use IRSA for barman tool.
    metadata:
      annotations:
        eks.amazonaws.com/role-arn: <iam-role-arn>

  backup:
    barmanObjectStore:
      destinationPath: s3://[s3-bucket-name]/[prefix]
      # Get the Temporary AWS credentails from IAM role.
      s3Credentials:
        inheritFromIAMRole: true
      wal:
        compression: gzip
        maxParallel: 8
    retentionPolicy: "7d"
```
Here, we use AWS IAM Roles for Service Accounts (IRSA) to securely grant S3 access without hardcoding credentials. Ensure the IAM role has the necessary permissions.

#### Scheduling Automated Backups 

For reliability, schedule automated backups using the ScheduledBackup CRD:

```yaml
---
apiVersion: postgresql.cnpg.io/v1
kind: ScheduledBackup
metadata:
  name: test-db-backup
  namespace: postgres
spec:
  schedule: "0 0 0 * * *" # Daily at midnight
  immediate: true
  backupOwnerReference: self
  cluster:
    name: test-db
```
Setting `spec.immediate: true` triggers an immediate backup upon creation.

#### On-Demand Backups

For ad-hoc backups, use the Backup CRD:

```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Backup
metadata:
  name: ondemand-example
  namespace: postgres
spec:
  method: barmanObjectStore
  cluster:
    name: test-db
```

### Recovery

CNPG supports several recovery methods, including Point-in-Time Recovery (PITR), which allows restoring a cluster to a specific timestamp between the earliest backup and the latest archived WAL file.
|
> Note:
> - PITR requires WAL archiving to be enabled.
> - CNPG does not support in-place recovery. Instead, you must bootstrap a new cluster from a backup.

#### Recovering from a Backup
1. List available backups:
```
$ kubectl get backup -n postgres
NAME                                AGE     CLUSTER       METHOD              PHASE       ERROR
test-db-backup-20250525000000       4h40m   test-db       barmanObjectStore   completed
```

2. Create a new Cluster resource to restore from the backup:
```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: example-recovery-cluster
  namespace: postgres
spec:
  instances: 3

  bootstrap:
    recovery:
      backup:
        name: test-db-backup-20250525000000

  storage:
    size: 2Gi
```

### Conclusion

Deploying PostgreSQL on Kubernetes with CloudNativePG (CNPG) unlocks a new era of managing stateful workloads in cloud-native environments. By leveraging Kubernetes-native tooling like CRDs, Helm, and Prometheus, CNPG simplifies:
- ✅ High-availability PostgreSQL clusters
- ✅ Automated backups and recovery (including PITR)
- ✅ Seamless monitoring and scaling

While traditional databases were once considered unfit for Kubernetes, CNPG proves that PostgreSQL can thrive—with the right operator. Whether you’re migrating an existing database or deploying a new one, CNPG offers a production-ready, declarative approach to database management.