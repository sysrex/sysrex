---
title: "Create EKS Cluster with Role Assumption"
date: 2019-08-02T14:16:40+01:00
tags: ["AWS", "EKS"]
summary: "How to create an EKS Cluster with role assumption"
draft: false
---



When creating an EKS Cluster you need to specify a role that is automatically granted system:masters permissions in the cluster's RBAC configuration. The role has be to created in the account where EKS will be created. If you like me are using _Assume Role_ form another account in order to avoid creating a local account the AWS account just create the role as you would normally do and then:



![Trust Relationship](/images/trust_relationship.png)



The policy JSON has to be edited like this:


{{<highlight json>}}

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "eks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    },
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<account:/role>"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}

{{</highlight>}}