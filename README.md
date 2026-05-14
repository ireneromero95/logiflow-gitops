# logiflow-gitops

![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=flat&logo=kubernetes&logoColor=white)
![KEDA](https://img.shields.io/badge/KEDA-purple?style=flat&logo=keda&logoColor=white)
![ArgoCD](https://img.shields.io/badge/ArgoCD-EF7B4D?style=flat&logo=argo&logoColor=white)
![AWS SQS](https://img.shields.io/badge/AWS_SQS-FF9900?style=flat&logo=amazonsqs&logoColor=white)
![AWS SNS](https://img.shields.io/badge/AWS_SNS-FF9900?style=flat&logo=amazonaws&logoColor=white)
![EKS](https://img.shields.io/badge/EKS-FF9900?style=flat&logo=amazoneks&logoColor=white)
![Helm](https://img.shields.io/badge/Helm-0F1689?style=flat&logo=helm&logoColor=white)

GitOps pipeline that scales Kubernetes workers based on SQS queue depth using KEDA and ArgoCD — with real-time email and SMS alerts via SNS.

---

## What this is

This repository contains the GitOps manifests for the **LogiFlow autoscaling pipeline** — a lab project simulating a real-world problem: a queue worker that can't keep up with overnight batch uploads.

Worker pods scale from **0 to N** based on the depth of an AWS SQS queue. KEDA drives the autoscaling logic, and ArgoCD ensures that every change goes through Git. No manual `kubectl apply` — Git is the only path to the cluster.

---

## Architecture

```
┌─────────────┐     uploads messages     ┌──────────────────┐
│  Warehouse  │ ──────────────────────►  │  AWS SQS Queue   │
│  Simulator  │                          │  (shipments)     │
└─────────────┘                          └────────┬─────────┘
                                                  │ queue depth metric
                                                  ▼
                                          ┌───────────────┐
                                          │     KEDA      │
                                          │  ScaledObject │
                                          └──────┬────────┘
                                                 │ drives replica count
                                                 ▼
                                      ┌─────────────────────┐
                                      │  Worker Deployment  │
                                      │  (0 → N pods)       │
                                      └──────────┬──────────┘
                                                 │
                                     ┌───────────┼───────────┐
                                     ▼           ▼           ▼
                                  Worker 1   Worker 2   Worker N

                                     when scaling happens:
                                                 │
                                                 ▼
                                     ┌───────────────────────┐
                                     │  Event Exporter       │
                                     └──────────┬────────────┘
                                                │
                                                ▼
                                     ┌───────────────────────┐
                                     │  SNS Notifier         │
                                     └──────────┬────────────┘
                                                │
                               ┌────────────────┴────────────────┐
                               ▼                                 ▼
                          Email                              SMS
```

**GitOps flow:**

```
  Your laptop        GitHub             ArgoCD            EKS cluster
       │                │                 │                    │
       ├── git push ──► │                 │                    │
       │                ├── webhook ────► │                    │
       │                │                 ├── sync ──────────► │
```

---

## Repository structure

```
logiflow-gitops/
├── apps/
│   └── worker/
│       ├── deployment.yaml          # Worker deployment + ConfigMap
│       ├── service-account.yaml     # IRSA-annotated service account
│       └── keda-scaledobject.yaml   # TriggerAuthentication + ScaledObject
├── infra/
│   └── alerting/
│       ├── event-exporter.yaml      # Watches k8s events and forwards them
│       └── sns-notifier.yaml        # Receives events and publishes to SNS
└── scripts/
    └── warehouse-simulator.sh       # Sends synthetic SQS load for testing
```

---

## Key concepts

**Why KEDA instead of HPA?**
The worker barely uses CPU — it mostly waits for I/O. HPA would see 3% CPU and do nothing, even with 50,000 messages in the queue. KEDA scales on queue depth directly, which is the signal that actually matters. It also supports scale-to-zero, so there are no running pods (and no cost) when the queue is empty.

**Why GitOps?**
ArgoCD enforces that the cluster always matches what's in Git. If someone manually changes a deployment, ArgoCD reverts it automatically. Every change has a Git history, a commit message, and can go through a pull request.

**Alert pipeline:**
Kubernetes generates events when scaling happens → Event Exporter catches them and sends them via HTTP → SNS Notifier publishes to an SNS topic → email or SMS lands in the on-call engineer's inbox.

---

## Stack

Kubernetes · KEDA · ArgoCD · AWS SQS · AWS SNS · IRSA · EKS · Helm
