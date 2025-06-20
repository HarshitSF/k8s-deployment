## Cluster Autoscaler Strategy for EKS

The Cluster Autoscaler (CA) automatically adjusts the number of nodes in your EKS cluster based on pending pods.

### EKS Setup Strategy

Use Auto Scaling Groups (ASG) with tags that Cluster Autoscaler can manage.

1. Tag your ASG with these keys:

```
k8s.io/cluster-autoscaler/enabled = true
k8s.io/cluster-autoscaler/<YOUR-CLUSTER-NAME> = owned
```

2. Install CA using Helm:

```
helm repo add autoscaler https://kubernetes.github.io/autoscaler
helm repo update

helm install cluster-autoscaler autoscaler/cluster-autoscaler \
  --namespace kube-system \
  --set autoDiscovery.clusterName=<your-cluster-name> \
  --set awsRegion=<your-region> \
  --set rbac.create=true \
  --set extraArgs.balance-similar-node-groups=true \
  --set extraArgs.skip-nodes-with-local-storage=false \
  --set extraArgs.expander=least-waste
```

3. Set IAM Permissions for CA to manage nodes (typically using an IAM role + trust relationship with eks.amazonaws.com).

### Benefits

1. Automatically provisions nodes when HPA scales pods

2. Scales down unused nodes

3. Optimizes cost and resource utilization