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
Create namespace 'artifactory'
```
kubectl create namespace artifactory
```

### Keys
Create MASTER_KEY or use existing one?
```
# Create a key
export MASTER_KEY=$(openssl rand -hex 32)
echo ${MASTER_KEY}
 
# Create a secret containing the key. The key in the secret must be named master-key
kubectl create secret generic my-masterkey-secret -n artifactory --from-literal=master-key=${MASTER_KEY}
``` 
When using Helm subsequently, pass the following:
```
--set artifactory.masterKeySecretName=my-masterkey-secret
```

Create JOIN_KEY
```
# Create a key
export JOIN_KEY=$(openssl rand -hex 32)
echo ${JOIN_KEY}
 
# Create a secret containing the key. The key in the secret must be named join-key
kubectl create secret generic my-joinkey-secret -n artifactory --from-literal=join-key=${JOIN_KEY}
```

When using Helm subsequently, pass the following:
```
--set artifactory.joinKeySecretName=my-joinkey-secret
```

### Helm
Add JFrog repo
```
helm repo add jfrog https://charts.jfrog.io
helm repo update
```

