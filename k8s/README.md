# Setting up the cluster
When we setup the cluster, we're still missing a few things:
- A service account for artifactory, with
- A role that enables access to EBS 

From JFrog doc


```
cluster_name=artifactory-eks-cluster
oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
echo $oidc_id
```
Add policy as described in AWS doc:
https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html


We will add them manually:
```
eksctl create iamserviceaccount \
        --name ebs-csi-controller-sa \
        --namespace kube-system \
        --cluster artifactory-eks-cluster \
        --role-name AmazonEKS_EBS_CSI_DriverRole \
        --role-only \
        --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
        --approve
```

Verify the default storage class:
```
$ kubectl get storageclass
NAME   PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
gp2    kubernetes.io/aws-ebs   Delete          WaitForFirstConsumer   false                  71m
```
If no class exists, or it's not the default, change it:
```
$ kubectl patch storageclass gp2 -p '{"metadata": {"annotations": {"storageclass.kubernetes.io/is-default-class": "true"}}}'
storageclass.storage.k8s.io/gp2 patched

$ kubectl get storageclass
NAME            PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
gp2 (default)   kubernetes.io/aws-ebs   Delete          WaitForFirstConsumer   false                  72m

```
Later we will override these when running helm for artifactory:
```
helm install artifactory jfrog/artifactory \
  --namespace artifactory \
  --set serviceAccount.create=false \
  --set serviceAccount.name=artifactory-sa
```


# Cleanup
```
kubectl scale statefulset artifactory --replicas=0 -n artifactory
helm uninstall artifactory -n artifactory
kubectl delete namespace artifactory
#eksctl delete cluster --name artifactory-eks-cluster
```
