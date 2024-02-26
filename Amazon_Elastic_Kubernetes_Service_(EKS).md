```
Step by step to create an eks cluster
https://katharharshal1.medium.com/deploying-a-kubernetes-cluster-with-amazon-eks-ae9f4ff18f77
Set up cluster autoscaler: 
https://katharharshal1.medium.com/kubernetes-cluster-autoscaling-ca-using-aws-eks-4aab8c89f9a1

```

# **Step 1: Creating an EKS role**

First, we create an IAM role for EKS.

- Open the IAM console, select ROLES on the left and then click the create role button at the top of the page.
- Select “AWS service” as the trusted entity type and “EKS” as the use case and choose EKS - Cluster and click NEXT as shown below.





- In the add permission section, the “AmazonEKSClusterPolicy” is automatically selected. Leave it as it and click NEXT
- Enter a name for the role (*`eksmasterrole`*) and hit the **Create role** button at the bottom of the page to create the IAM role.


2. Create a VPC to deploy the cluster
Go to “AWS CloudFormation” and click on “Create Stack”; under prepare template section, click “Template is ready” and enter below URL as “Amazon S3 URL”.

```
https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2019-02-11/amazon-eks-vpc-sample.yaml

```

Give the stack name eksvpc and after reviewing the CIDR, Subnet. Click on create a stack

# **Output:**

After the stack is completed, in the output session you got a Security GroupId, VpcId, SubnetId. **This information is useful when creating an EKScluster.**