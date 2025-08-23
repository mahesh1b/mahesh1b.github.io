---
date: '2025-08-22T09:54:26+05:30'
draft: false
title: 'Vault Policy'
tags: ["Hashicorp", "Vault", "hcl", "access"]
---


```hcl
# Example policy
path "secret/data/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

path "secret/data/prod/*" {
  capabilities = ["deny"]
}

path "sys/mounts" {
  capabilities = ["read", "list"]
}
```