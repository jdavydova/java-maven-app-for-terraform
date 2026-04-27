# 🚀 EKS Cluster Autoscaling Setup

This guide explains how to configure **Kubernetes Cluster Autoscaler** with **AWS Auto Scaling Groups (ASG)** in an Amazon EKS cluster.

---

## 📌 Overview

Amazon EKS does **not automatically scale EC2 worker nodes**.

To enable autoscaling, you must:

- Use an **Auto Scaling Group (ASG)** for EC2 instances
- Deploy the **Kubernetes Cluster Autoscaler**
- Configure **IAM + OIDC trust** between AWS and Kubernetes

---

## 🏗️ 1. Create VPC Infrastructure

Deploy the required networking stack using the AWS CloudFormation template:

https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml

---

## ⚙️ 2. Understand Auto Scaling Groups (ASG)

An Auto Scaling Group:

- Groups EC2 instances logically
- Defines scaling boundaries:
  - **Min size** → minimum number of nodes
  - **Max size** → maximum number of nodes

> ⚠️ Important  
> ASG **does not automatically scale based on Kubernetes workloads**

---

## 🤔 3. Why Autoscaler Is Required

EKS alone does not manage scaling.

You must deploy the **Cluster Autoscaler**, which:

- Watches for unscheduled pods
- Increases node count when needed
- Decreases node count when underutilized

---

## 🔐 4. Configure OIDC Trust (AWS ↔ Kubernetes)

To allow Kubernetes to control AWS resources securely, configure **OIDC (OpenID Connect)**.

### What is OIDC?

- Identity layer on top of OAuth 2.0
- Allows Kubernetes ServiceAccounts to assume IAM roles

---

### 🔍 4.1 Get OIDC Provider URL

```bash
aws eks describe-cluster   --name <cluster-name>   --query "cluster.identity.oidc.issuer"   --output text
```

---

### ➕ 4.2 Create IAM OIDC Identity Provider

1. Go to **AWS Console → IAM → Identity Providers**
2. Click **Add provider**
3. Select:
   - **Type**: OpenID Connect
4. Enter:
   - **Provider URL** → (EKS OIDC URL)
   - **Audience** → `sts.amazonaws.com`
5. Save

---

## 📜 5. Create IAM Policy

Create a policy named:

ClusterAutoscalerPolicy

### Policy JSON

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "autoscaling:DescribeAutoScalingGroups",
        "autoscaling:DescribeAutoScalingInstances",
        "autoscaling:DescribeLaunchConfigurations",
        "autoscaling:DescribeScalingActivities",
        "autoscaling:SetDesiredCapacity",
        "autoscaling:TerminateInstanceInAutoScalingGroup",
        "ec2:DescribeInstanceTypes",
        "ec2:DescribeLaunchTemplateVersions"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
```

---

## 👤 6. Create IAM Role for Autoscaler

1. Go to **IAM → Roles → Create role**
2. Select:
   - **Web identity**
3. Choose:
   - Your OIDC provider
4. Set:
   - Audience → `sts.amazonaws.com`
5. Attach:
   - `ClusterAutoscalerPolicy`

---

## ☸️ 7. Create Kubernetes ServiceAccount

Bind the IAM role to Kubernetes using annotations:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  annotations:
    eks.amazonaws.com/role-arn: <IAM-role-arn>
```

---

## 🚀 8. Deploy Cluster Autoscaler

Deploy the Cluster Autoscaler to your cluster.

It will:

- Monitor pod scheduling
- Increase nodes when pods cannot be scheduled
- Decrease nodes when underutilized

---

## 🔑 Summary

| Component              | Responsibility                          |
|----------------------|------------------------------------------|
| **ASG**              | Defines EC2 scaling limits               |
| **Cluster Autoscaler** | Makes scaling decisions                 |
| **OIDC + IAM**       | Enables secure AWS access from Kubernetes |

---

## 🎯 Result

With everything configured:

✅ Pods trigger scaling automatically  
✅ EC2 instances scale up/down dynamically  
✅ Infrastructure cost is optimized  
