---
title: "EC2 backup with lambda"
date: 2019-04-11T19:16:40+01:00
tags: ["AWS", "Code"]
summary: "Create ec2 backup with lambda and python functions"
draft: false
---


#Create EC2 Snapshots with AWS Lambda and Python Functions



### Create the IAM Role that will be used

First in IAM, navigate to Services -> IAM -> Roles -> Create Role. Then select Lambda and create a policy for the role.

We want our function to be able to:

- Read different types of information from EC2 (so we’ll give it full describe and read permissions)

- Create and Delete Snapshots

- Create and Delete Tags

- Write access to CloudWatch Logs

{{< highlight json >}}

{
   "Version": "2012-10-17",
   "Statement": [
       {
           "Effect": "Allow",
           "Action": "logs:*",
           "Resource": "*"
       },
       {
           "Effect": "Allow",
           "Action": "ec2:Describe*",
           "Resource": "*"
       },
       {
           "Effect": "Allow",
           "Action": [
               "ec2:CreateSnapshot",
               "ec2:DeleteSnapshot",
               "ec2:CreateTags",
               "ec2:DeleteTags",
               "ec2:ModifySnapshotAttribute"
           ],
           "Resource": [
               "*"
           ]
       }
   ]
}

{{< /highlight >}}

### Tagging

To label which EC2 Instances we want to snapshot – we will use tags.

Simply select the instance which we want to automatically snapshot – then give it the tag Key: ‘auto_snapshot’ and Value: 'true' .


### Lambda

Now, navigate to the AWS Lambda Management Console. Then select Create Function > Author from Scratch. Name your function, choose Python 3.6 as the runtime, finally for roles select Choose an Existing Role (and select the role we made earlier).


### Python 3.6 script to be added to the function, gist here: [(Github)](https://gist.github.com/sysrex/f07d012797c681f69410150e094c7296)

{{< highlight python >}}

import collections
import datetime
import time
import sys


today = datetime.date.today()
today_string = today.strftime('%Y/%m/%d')
delete_after_days = 2  # Delete snapshots after this many days

# Except after Monday (at Tuesday ~1am), since Friday is only 2 'working' days away:
if datetime.date.today().weekday() == 1:
    delete_after_days = delete_after_days + 2

deletion_date = today - datetime.timedelta(days=delete_after_days)
deletion_date_string = deletion_date.strftime('%Y/%m/%d')


ec2 = boto3.client('ec2')
regions = ec2.describe_regions().get('Regions',[] )
all_regions = [region['RegionName'] for region in regions]

def lambda_handler(event, context):
    snapshot_counter = 0
    snap_size_counter = 0
    deletion_counter = 0
    deleted_size_counter = 0

  for region_name in all_regions:
      print('Instances in EC2 Region {0}:'.format(region_name))
      ec2 = boto3.resource('ec2', region_name=region_name)

      # We only want to look through instances with the following tag key value pair: auto_snapshot : true
      instances = ec2.instances.filter(
          Filters=[
              {'Name': 'tag:auto_snapshot', 'Values': ['true']}
                  ]
              )

      volume_ids = []
      for i in instances.all():

          for tag in i.tags:  # Get the name of the instance
              if tag['Key'] == 'Name':
                  name = tag['Value']

          print('Found tagged instance \'{1}\', id: {0}, state: {2}'.format(i.id, name, i.state['Name']))

          vols = i.volumes.all()  # Iterate through each instance's volumes
          for v in vols:
              print('{0} is attached to volume {1}, proceeding to snapshot'.format(name, v.id))
              volume_ids.extend(v.id)
              snapshot = v.create_snapshot(
                  Description = 'AutoSnapshot of {0}, on volume {1} - Created {2}'.format(name, v.id, today_string),
                  )
              snapshot.create_tags(  # Add the following tags to the new snapshot
                  Tags = [
                      {
                          'Key': 'auto_snap',
                          'Value': 'true'
                      },
                      {
                          'Key': 'volume',
                          'Value': v.id
                      },
                      {
                          'Key': 'CreatedOn',
                          'Value': today_string
                      },
                       {
                          'Key': 'Name',
                          'Value': '{} autosnap'.format(name)
                      }
                  ]
              )
              print('Snapshot completed')
              snapshot_counter += 1
              snap_size_counter += snapshot.volume_size

              # Now iterate through snapshots which were made by autsnap
              snapshots = ec2.snapshots.filter(
                  Filters=[
                      {'Name': 'tag:auto_snap', 'Values': ['true']
                      }
                  ]
              )


              print('Checking for out of date snapshots for instance {0}...'.format(name))
              for snap in snapshots:
                  can_delete = False
                  for tag in snap.tags: # Use these if statements to get each snapshot's
                                        # cleated on date, name and auto_snap tag
                      if tag['Key'] == 'CreatedOn':
                          created_on_string = tag['Value']
                      if tag['Key'] == 'auto_snap':
                          if tag['Value'] == 'true':
                              can_delete = True
                      if tag['Key'] == 'Name':
                          name = tag['Value']
                  created_on = datetime.datetime.strptime(created_on_string, '%Y/%m/%d').date()

                  if created_on <= deletion_date and can_delete == True:
                      print('Snapshot id {0}, ({1}) from {2} is {3} or more days old... deleting'.format(snap.id, name, created_on_string, delete_after_days))
                      deleted_size_counter += snap.volume_size
                      snap.delete()
                      deletion_counter += 1

  print('   Made {0} snapshots totalling {1} GB\
        Deleted {2} snapshots totalling {3} GB'.format(snapshot_counter, snap_size_counter, deletion_counter, deleted_size_counter))
  return


{{< /highlight >}}



This is just an improvement and personal notes to:
[(Keyrus Blog)](http://blog.keyrus.co.uk/backup_ec2_instances_automatic_snapshots.html)

