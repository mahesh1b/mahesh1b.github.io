---
date: '2025-08-22T09:44:30+05:30'
draft: false
title: 'Gitlab CI'
tags: ["Gitlab", "CICD", "yaml"]
---

```yaml
stages:          # Pipeline stages (run sequentially)
  - build
  - test
  - deploy

variables:       # Define environment variables
  NODE_ENV: test

build_job:       # Job name
  stage: build   # Which stage it belongs to
  script:        # Commands to run
    - npm install

test_job:
  stage: test
  script:
    - npm test

deploy_job:
  stage: deploy
  script:
    - echo "Deploying..."
  only:
    - main       # Run only on main branch
```
