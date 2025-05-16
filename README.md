# Artifactory dep

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
#### v7.77.3
- Helm chart version is 107.77.3 
```
helm install artifactory jfrog/artifactory --version "107.77.3" -f custom-values.yaml -n artifactory --create-namespace

helm upgrade --install artifactory jfrog/artifactory --version "107.77.3" -f custom-values.yaml -n artifactory --create-namespace 

```
Verify pod versions:
```
kubectl get pods -n artifactory  -o jsonpath='{.items[*].spec.containers[*].image}' 
```


#### Cleanup
```
kubectl scale statefulset artifactory --replicas=0 -n artifactory
# make sure to delete pvcs and pvs for a clean slate
helm uninstall artifactory && sleep 90 && kubectl delete pvc -l app=artifactory
kubectl delete namespace artifactory

# if you want to remove the cluster
#eksctl delete cluster --name artifactory-eks-cluster
```



### Latest version
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
helm upgrade --install jfrog/artifactory --namespace artifactory --create-namespace jfrog/artifactory


```

#### Upgrade Artifactory 
```
helm upgrade artifactory jfrog/artifactory -f custom-values.yaml -n artifactory
```

