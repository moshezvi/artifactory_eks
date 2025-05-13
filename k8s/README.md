# Create the cluster
To create the cluster, run:
```
eksctl create cluster -f artifactory-cluster.yaml
```
# Additions - EKS specific
## Service account and role
When we setup the cluster, we're still missing a few things:
- A service account for artifactory, with
- A role that enables access to EBS 

<!-- ```
# Verify OIDC is configured properly
cluster_name=artifactory-eks-cluster
oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
echo $oidc_id
``` -->
Create Service Account and policy as described in AWS doc:
https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html

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

## Default storage class
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

