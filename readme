# Calico Staged Network Policies Demo on AKS

This guide demonstrates how to deploy Calico on AKS with staged network policies using the YaoBank application.

## Prerequisites

- AKS cluster with Azure CNI
- kubectl configured to access your cluster
- Helm 3.x installed

## Step 1: Install Calico on AKS

### 1.1 Add the Project Calico Helm repository
```bash
helm repo add projectcalico https://docs.projectcalico.org/charts
```

### 1.2 Update Helm repositories
```bash
helm repo update
```

### 1.3 Install Calico using the tigera-operator chart
```bash
helm install calico projectcalico/tigera-operator \
  --version v3.30.0 \
  --namespace tigera-operator \
  --create-namespace \
  --values calico-helm-values/calico-values.yaml
```

### 1.4 Verify Calico installation
```bash
# Check if tigera-operator is running
kubectl get pods -n tigera-operator

# Wait for Calico components to be ready
kubectl wait --for=condition=Ready pods --all -n calico-system --timeout=300s

# Verify network policy is now enabled
az aks show --resource-group calicooss --name calico-whisker --query "networkProfile.networkPolicy" -o tsv
```

## Step 2: Deploy the YaoBank Application

### 2.1 Deploy the application
```bash
kubectl apply -f yaobank.yaml
```

### 2.2 Verify application deployment
```bash
# Check all pods are running
kubectl get pods -n yaobank

# Wait for all deployments to be ready
kubectl wait --for=condition=Available deployment --all -n yaobank --timeout=300s
```

## Step 3: Apply Staged Network Policies

### 3.1 Apply the staged network policies
```bash
# Apply all staged network policies from the dedicated folder
kubectl apply -f staged-network-policy/
```

### 3.2 Verify staged policies are created
```bash
# List all staged network policies in the yaobank namespace
kubectl get stagednetworkpolicies -n yaobank

# Get detailed information about the policies
kubectl describe stagednetworkpolicies -n yaobank
```

## Step 4: Monitor Staged Network Policies

### 4.1 Query Whisker backend for staged policy effects
```bash
# Query flows that would be denied by staged network policies
curl -G 'http://127.0.0.1:8081/whisker-backend/flows' \
  --data-urlencode startTimeGte=-60 \
  --data-urlencode startTimeLt=0 | \
  jq '.items[] | select(.policies.pending != null and (any(.policies.pending[]; .action == "Deny" and .trigger.kind == "StagedNetworkPolicy")))'
```


## Step 5: Promote Staged Policies to Enforced

### 5.1 Apply enforced network policies
```bash
# Apply the enforced network policies from the dedicated folder
kubectl apply -f network-policy/

# Remove the staged policies after applying enforced ones
kubectl delete -f staged-network-policy/
```

### 5.2 Test enforced policies


## Cleanup

### Uninstall everything
```bash
# Remove the application
kubectl delete -f yaobank.yaml

# Remove staged network policies
kubectl delete -f staged-network-policy/

# Remove any enforced network policies (if converted)
kubectl delete networkpolicies --all -n yaobank

# Uninstall Calico
helm uninstall calico -n tigera-operator
kubectl delete namespace tigera-operator
```

## Troubleshooting

### Common Issues

1. **Pods stuck in pending**: Check if Calico components are ready
   ```bash
   kubectl get pods -n calico-system
   kubectl get pods -n tigera-operator
   ```

2. **Network policies not working**: Verify Calico is properly installed
   ```bash
   kubectl get installation default -o yaml
   ```

3. **Flow logs not appearing**: Ensure proper annotations and Calico Enterprise features are enabled

### Useful Commands
```bash
# Check Calico node status
kubectl get nodes -o wide

# View Calico configuration
kubectl get installation default -o yaml

# Check for policy violations in logs
kubectl logs -n calico-system -l k8s-app=calico-node --tail=100 | grep -i deny
```

## Additional Resources

- [Calico Documentation](https://docs.projectcalico.org/)
- [Staged Network Policies Guide](https://docs.projectcalico.org/security/staged-network-policies)
- [AKS with Calico](https://docs.microsoft.com/en-us/azure/aks/use-network-policies)