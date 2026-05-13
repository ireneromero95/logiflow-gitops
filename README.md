![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=flat&logo=kubernetes&logoColor=white)
![KEDA](https://img.shields.io/badge/KEDA-purple?style=flat&logo=keda&logoColor=white)
![ArgoCD](https://img.shields.io/badge/ArgoCD-EF7B4D?style=flat&logo=argo&logoColor=white)
![AWS SQS](https://img.shields.io/badge/AWS_SQS-FF9900?style=flat&logo=amazonsqs&logoColor=white)
![AWS SNS](https://img.shields.io/badge/AWS_SNS-FF9900?style=flat&logo=amazonaws&logoColor=white)
![EKS](https://img.shields.io/badge/EKS-FF9900?style=flat&logo=amazoneks&logoColor=white)

This repository contains the GitOps manifests for the LogiFlow autoscaling pipeline.

Worker pods scale from 0 to N based on the depth of an AWS SQS queue, using KEDA as the autoscaler and ArgoCD to enforce that every change goes through Git. No manual kubectl apply — Git is the only path to the cluster.

What's included:
  • KEDA ScaledObject — scales on queue depth, not CPU
  • ArgoCD Applications — automated sync with self-healing
  • SNS Notifier + Event Exporter — email/SMS alerts on scaling events
  • Warehouse Simulator — sends synthetic SQS load for testing

Stack: Kubernetes · KEDA · ArgoCD · AWS SQS · AWS SNS · IRSA · EKS
