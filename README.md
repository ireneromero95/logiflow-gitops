This repository contains the GitOps manifests for the LogiFlow autoscaling pipeline.

Worker pods scale from 0 to N based on the depth of an AWS SQS queue, using KEDA as the autoscaler and ArgoCD to enforce that every change goes through Git. No manual kubectl apply — Git is the only path to the cluster.

What's included:
  • KEDA ScaledObject — scales on queue depth, not CPU
  • ArgoCD Applications — automated sync with self-healing
  • SNS Notifier + Event Exporter — email/SMS alerts on scaling events
  • Warehouse Simulator — sends synthetic SQS load for testing

Stack: Kubernetes · KEDA · ArgoCD · AWS SQS · AWS SNS · IRSA · EKS
