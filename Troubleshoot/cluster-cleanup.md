# Manual Cleanup Procedure for Stuck EKS VPC
Eventhough cluster setup deleted via CLI aws cosole showing failed

![Cluster deletion failed](Troubleshoot/cluster-deletion-failed.png)

### Step 1: Identify Dependencies on the Subnets

✅ Go to VPC → Subnets:

- Find subnet-00488b23d9601f63e and subnet-086567a5bda58cf79.

- Click on each subnet → Resources tab → Check what is still using it.

Look for:

- ENIs (Network Interfaces) → Delete them.

- EC2 Instances → Terminate them.

### Step 2: Delete Dependent Resources
✅ LoadBalancer

Go to Load Balancers 

- Delete them if present.

![Delete LB](Troubleshoot/delete-lb.png)

### Step 3: Delete the VPC 
✅ Go to VPC

- Delete teh vpc which is attached to eksctl

![Delete VPC](Troubleshoot/delete-vpc.png)

### Step 4: Retry Stack Deletion

Once everything is manually detached/deleted:

Go back to CloudFormation.

Click Retry delete.

It should now delete the VPC and finish successfully.

Step 4: Verify Cleanup

Ensure the VPC no longer exists under VPC → Your VPCs.

Ensure no leftover ENIs, route tables, subnets, or security groups are present.
