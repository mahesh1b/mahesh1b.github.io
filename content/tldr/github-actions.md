---
date: '2025-08-22T09:22:13+05:30'
draft: false
title: 'Github Actions'
tags: ["Github", "CICD", "yaml"]
---


```yaml
name: CI Pipeline   # Name of the workflow

on:                 # Triggers (events)
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:               # Workflow is made of jobs
  build:
    runs-on: ubuntu-latest   # VM/runner
    steps:                   # Steps in this job
      - name: Checkout repo
        uses: actions/checkout@v4  # Reusable action

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test
```