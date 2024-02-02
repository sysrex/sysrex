---
title: "Terraform with Remote S3 and Locking state"
date: 2019-11-04T14:16:40+01:00
tags: ["AWS", "Terraform"]
summary: "Terraform with AWS and remote state bucket"
draft: false
---

  Ok .... so you .... you need to build infrastructure for your team. This is a job for terraform. Whenever you execute your plans in terraform a "state" file gets created called *terraform.tfstate*,
this generally is created in the .terraform directory and contains information about the infrastructure and configuration that terraform is managing. It is generally accepted that when multiple people work on the same project it's better to store this state file remotely so that more members of the team can access it to make changes to the infrastructure, I would dare to add even if you use multiple computers this is a wise thing to do.


The state file contains information about what real resources exist for each object defined in the terraform config files. For example, if you have a DNS zone resource created in your terraform config, then the state file contains info about the actual resource that was created on AWS.



Here is an example of creating a DNS zone with Terraform:


{{< highlight terraform >}}
resource "aws_route53_zone" "example_dns_zone" {
  name = "domain.com"
}
{{< /highlight >}}

And this is how a state file looks like:

{{< highlight terraform >}}
"aws_route53_zone.example_dns_zone": {
    "type": "aws_route53_zone",
    "primary": {
       "id": "Z2D2ARTZHH4NUA",
       "attributes": {       
          "name": "domain.com"
        }
     }
},
{{< /highlight >}}



### Store State Remotely in S3

If you are working on a team, then its best to store the terraform state file remotely so that many people can access it. In order to setup terraform to store state remotely you need two things: an s3 bucket to store the state file in and an terraform s3 backend resource.

You can create an s3 bucket in a terraform config like so:


{{< highlight terraform >}}
provider "aws" {
  region = "us-west-2"
}
{{< /highlight >}}

{{< highlight terraform >}}
resource "aws_s3_bucket" "terraform-state-storage-s3" {
    bucket = "terraform-remote-state-storage-s3"
 
    versioning {
      enabled = true
    }
 
    lifecycle {
      prevent_destroy = true
    }
 
    tags {
      Name = "S3 Remote Terraform State Store"
    }      
}
{{< /highlight >}}
<br>

Then create the s3 backend resource like so:

<br>

{{< highlight terraform >}}
terraform {
 backend “s3” {
 encrypt = true
 bucket = "terraform-remote-state-storage-s3"
 region = us-west-2
 key = path/to/state/file
 }
}
{{< /highlight >}}

What is locking and why do we need it?

If the state file is stored remotely so that many people can access it, then you risk multiple people attempting to make changes to the same file at the exact same time. So we need to provide a mechanism that will “lock” the state if its currently in-use by another user. We can accomplish this by creating a dynamoDB table for terraform to use.

Create the dynamoDB table like this:  
<br>

{{< highlight terraform >}}
resource "aws_dynamodb_table" "dynamodb-terraform-state-lock" {
  name = "terraform-state-lock-dynamo"
  hash_key = "LockID"
  read_capacity = 20
  write_capacity = 20
 
  attribute {
    name = "LockID"
    type = "S"
  }
 
  tags {
    Name = "DynamoDB Terraform State Lock Table"
  }
}
{{< /highlight >}}

<br>

You will need to modify the Terraform S3 backend resource and add in the dynamoDB table:

<br>

{{< highlight terraform >}}
terraform {
 backend “s3” {
 encrypt = true
 bucket = "terraform-remote-state-storage-s3"
 dynamodb_table = "terraform-state-lock-dynamo"
 region = us-west-2
 key = path/to/state/file
 }
}
{{< /highlight >}}

<br>
Let's mix things up:

Once you’ve created the S3 bucket and dynamoDB table, along with the backend S3 resource referencing those, then you can run your terraform configs like normal with terraform plan and terraform apply commands and the state file will show up in the s3 bucket. After those commands, if you inspect .terraform/terraform.tfstate, you will see that it contains the location of the state file now instead of the actual state file.  
<br>

{{< highlight bash >}}
cat .terraform/terraform.tfstate
{{< /highlight >}}  
<br>


{{< highlight terraform >}}
{
    "version": 3,
    "backend": {
        "type": "s3",
        "config": {
            "bucket": "terraform-remote-state-storage-s3",
            "dynamodb_table": "terraform-state-lock-dynamo",
            "encrypt": true,
            "key": "example/terraform.tfstate",
            "region": "us-west-2"
        }
    }
}
{{< /highlight >}}
