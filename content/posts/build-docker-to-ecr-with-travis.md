---
title: "Build docker to ECR with travis"
date: 2019-02-07T14:19:40+01:00
tags: ["AWS"]
summary: "How to build docker containers and deploy them to ECR with Travis CI"
draft: false
---

You will need an AWS IAM Policy for the Travis user that looks pretty much like this _I strongly suggest you use a different user for this_

{{<highlight json>}}

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "ecr:PutLifecyclePolicy",
                "ecr:GetLifecyclePolicyPreview",
                "ecr:CreateRepository",
                "ecr:GetDownloadUrlForLayer",
                "ecr:GetAuthorizationToken",
                "ecr:ListTagsForResource",
                "ecr:UploadLayerPart",
                "ecr:ListImages",
                "ecr:DeleteLifecyclePolicy",
                "ecr:DeleteRepository",
                "ecr:PutImage",
                "ecr:UntagResource",
                "ecr:SetRepositoryPolicy",
                "ecr:BatchGetImage",
                "ecr:CompleteLayerUpload",
                "ecr:DescribeImages",
                "ecr:TagResource",
                "ecr:DescribeRepositories",
                "ecr:StartLifecyclePolicyPreview",
                "ecr:InitiateLayerUpload",
                "ecr:BatchCheckLayerAvailability",
                "ecr:GetRepositoryPolicy",
                "ecr:GetLifecyclePolicy"
            ],
            "Resource": "*"
        }
    ]
}

{{</highlight>}}


Next you will encrypt the created credentials in the .travis.yml file


{{<highlight bash>}}

travis encrypt AWS_ACCESS_KEY_ID=super_secret --add
travis encrypt AWS_SECRET_ACCESS_KEY=super_secret --add

{{</highlight>}}


Then your travis file should look like this


{{<highlight yaml>}}

dist: xenial
language: minimal
services:
  - docker

before_install:
  - pip install --user awscli
  - export PATH=$PATH:$HOME/.local/bin
script:
  - eval $(aws ecr get-login --no-include-email --region us-east-1)
  - make build-production
  - make push-image
env:
  global:
  - secure: <your encrypted key>
  - secure: <your encrypted key>


{{</highlight>}}