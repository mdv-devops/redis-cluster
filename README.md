### 1. Deploy Redis cluster

# Add helm chart into local repository list
```bash
helm repo add ot-helm https://ot-container-kit.github.io/helm-charts
```

# Update helm repository
```bash
helm repo update
```

# Install redis operator
```bash
helm install redis-operator ot-helm/redis-operator \
--values values.yaml  --namespace redis-operator --create-namespace
```

# Create secret for redis
```bash
kubectl create secret generic redis-secret --from-literal=password=Hh9QVEmdfcbHeZIGtU1n -n redis-operator
```

# Install redis cluster
```bash
helm upgrade redis-cluster ot-helm/redis-cluster \
--set redisCluster.clusterSize=3 --install --namespace redis-operator
```

# Verify the cluster by checking the pod status of leader and follower pods
```bash
watch kubectl get pod -n redis-operator
```

# Check the health of the redis cluster by using redis-cli
```bash
kubectl exec -it redis-cluster-leader-0 -n redis-operator -- redis-cli \
-a Hh9QVEmdfcbHeZIGtU1n cluster nodes
```

### Failover testing

# Write the dummy data using the redis-cli
```bash
kubectl exec -it redis-cluster-leader-0 -n redis-operator -- redis-cli \
-a Hh9QVEmdfcbHeZIGtU1n -c set ping pong
```

# Verify the key has been inserted properly by fetching its value
```bash
kubectl exec -it redis-leader-0 -n redis-operator -- redis-cli \
-a Hh9QVEmdfcbHeZIGtU1n -c get ping
```

# Delete the pod name redis-leader-0
```bash
kubectl delete pod redis-leader-0 -n redis-operator
```

# Try again to list redis cluster nodes from redis-leader-0
```bash
kubectl exec -it redis-cluster-leader-0 -n redis-operator -- redis-cli \
-a Hh9QVEmdfcbHeZIGtU1n cluster nodes
```

# Add Redis cluster dashboard into Grafana
```bash
kubectl apply -f configMap-for-grafana-dashboard/configmap.yaml
```
