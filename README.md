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
Create namespace 'artifactor
```
kubectl create namespace artifactory
```

### Keys
```
# Create keys and store them in kube
kubectl create secret generic artifactory-master-key -n artifactory --from-literal=master-key=$(openssl rand -hex 32)
kubectl create secret generic artifactory-join-key -n artifactory --from-literal=join-key=$(openssl rand -hex 32)

# add them to values.yaml, or use in helm calls as follows
--set artifactory.masterKeySecretName=artifactory-master-key
--set artifactory.joinKeySecretName=artifactory-join-key
```

### Helm
Add JFrog repo
```
helm repo add jfrog https://charts.jfrog.io
helm repo update
--set artifactory.masterKeySecretName=artifactory-master-key
--set artifactory.joinKeySecretName=artifactory-join-key

# OR
add the file flag --CHECK WHAT IT IS--

```

