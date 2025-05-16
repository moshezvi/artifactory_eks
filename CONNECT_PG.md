# Connecting Artifactory to PostGres

## Working with AWS VPC peering

1. Find the VPC ids and create a peering connection
```
* pg vpc id: vpc-0f49cc743f77c8359
* artifactory vpc id: vpc-0f40135447851080b

$ aws ec2 create-vpc-peering-connection --vpc-id vpc-0f40135447851080b --peer-vpc-id vpc-0f49cc743f77c8359

* peering-id: pcx-05c097eb5524fd46d

$ aws ec2 accept-vpc-peering-connection --vpc-peering-connection-id pcx-05c097eb5524fd46d
```

2. Find the route tables for artifactory
```
$ aws ec2 describe-route-tables --filters "Name=vpc-id,Values=<your-vpc-id>" --query "RouteTables[*].[RouteTableId,VpcId]" --output table
# e.g.
$ aws ec2 describe-route-tables --filters "Name=vpc-id,Values=vpc-0f49cc743f77c8359" --query "RouteTables[*].[RouteTableId,VpcId]" --output table

# find subnet associations as well - can help with identifying the correct route tables
$ aws ec2 describe-route-tables --filters "Name=vpc-id,Values=vpc-0f49cc743f77c8359" --query "RouteTables[*].{RouteTableId:RouteTableId, SubnetIds:Associations[*].SubnetId}" --output json

* artifactory route tables:  [rtb-04f740ef232b9dd9d, rtb-00be52806fc7253bd, rtb-0183b9843c2dd04f6]
```

3. In the artifactory Routing Tables, add a route to the PostGres cidr block through the peer connection
``` 
$ aws ec2 create-route --route-table-id <artifactory-route-table-id> --destination-cidr-block <postgres-vpc-cidr> --vpc-peering-connection-id <peering-id>

e.g., 
$ aws ec2 create-route --route-table-id rtb-04f740ef232b9dd9d --destination-cidr-block 10.0.0.0/16 --vpc-peering-connection-id pcx-05c097eb5524fd46d
```

CHECK THIS
4. Allow PG security group 
aws ec2 authorize-security-group-ingress --group-id <rds-security-group-id> --protocol tcp --port 5432 --cidr <artifactory-vpc-cidr>



====
## Pointing the deployment to the PG DB
1. Stop the service 
```
kubectl exec -it artifactory-0 -n artifactory -c artifactory -- /bin/bash
# gently stop the service
/opt/jfrog/artifactory/app/bin/artifactory.sh stop 

```
2. Go to the $JFROG_HOME dir (/opt/jfrog/):
```
$ cd /opt/jfrog/artifactory/var/etc/
# make a backup of the config file:
$ cp system.yaml system.yaml.bak
```

3. Copy the system.yaml file locally:
```
$ kubectl cp --container=artifactory <namespace>/<pod-name>:<container-path-to-file> <local-destination>
e.g., 
$ kubectl cp --container=artifactory artifactory/artifactory-0:/opt/jfrog/artifactory/var/etc/system.yaml ./system.yaml
```
edit the file, and then copy it back:
```
kubectl cp --container=artifactory ./system.yaml artifactory/artifactory-0:/opt/jfrog/artifactory/var/etc/system.yaml
```



## DEBUG
1. Verify artifactory can actually see the PG server:
```
kubectl exec -it artifactory-0 -n artifactory -c artifactory -- sh -c "getent hosts artifactory-pg-12.crco86gkgsng.us-east-2.rds.amazonaws.com"
``` 
2. Verify the pg security group allows access from artifactory 
