# Artifactory dep
## EKS cluster
To create the cluster, run:
```
eksctl create cluster -f artifactory-cluster.yaml
```

To verify the storageclass is set to default:
```
kubectl get storageclass
```
should return:
```
NAME   PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
gp2    kubernetes.io/aws-ebs   Delete          WaitForFirstConsumer   false                  37m
```

To make it default:
```
kubectl patch storageclass gp2 -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```
Then check the storage class:
```
NAME            PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
gp2 (default)   kubernetes.io/aws-ebs   Delete          WaitForFirstConsumer   false                  45ms
```

## Artifactory via Helm


### Keys
```
# Create namespace 'artifactory'
kubectl create namespace artifactory

# Create keys and store them in kube
kubectl create secret generic artifactory-master-key -n artifactory --from-literal=master-key=$(openssl rand -hex 32)
kubectl create secret generic artifactory-join-key -n artifactory --from-literal=join-key=$(openssl rand -hex 32)

# add them to custom-values.yaml, or use in helm calls as follows
--set artifactory.masterKeySecretName=artifactory-master-key
--set artifactory.joinKeySecretName=artifactory-join-key
```

### Helm
#### Add JFrog repo
```
helm repo add jfrog https://charts.jfrog.io
helm repo update
```

#### Install Artifactory
```
helm install artifactory artifactory \
  --set artifactory.masterKeySecretName=artifactory-master-key \
  --set artifactory.joinKeySecretName=artifactory-join-key

```
If using the custom-values file:
```
helm install artifactory jfrog/artifactory -f custom-values.yaml -n artifactory --create-namespace
```

#### Upgrade Artifactory 
```
helm upgrade artifactory jfrog/artifactory -f custom-values.yaml -n artifactory
```

# Cleanup
```
kubectl scale statefulset artifactory --replicas=0 -n artifactory
helm uninstall jfrog/artifactory -n artifactory

# make sure to delete pvcs and pvs for a clean slate

kubectl delete namespace artifactory
#eksctl delete cluster --name artifactory-eks-cluster
```


# v7.77.3
- Helm chart version is 107.77.3 
```
helm install artifactory jfrog/artifactory --version "107.77.3" -f custom-values.yaml -n artifactory 
```
Verify pod versions:
```
kubectl get pods -n artifactory  -o jsonpath='{.items[*].spec.containers[*].image}' 
```
