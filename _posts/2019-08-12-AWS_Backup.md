---
title: AWS -- EC2 backup with boto3 API
---

## Background

> We need to backup core ec2 instance for DR. We can run a script to call AWS API, it check the tag of instances to decide if need to take snapshot for backup. And clean up the expired snapshot.

## Steps

1. Set up tags for the instances which need to backup, example : key -- backup  value -- 7. This means backup the server every day and keep 7 snapshot at most.

2. Run the snapshot scripts, the scripts can be run in docker, here is a sample logic:

```python
import boto3
import copy
import os

from datetime import datetime, timedelta
from pprint import pprint

# Keep 3 Days of snapshot by default
DEFAULT_KEEP_DAYS = 3
# Not reboot the VM when take snapshoot
NOREBOOT = True


class Backup():
    def __init__(self, region_name, aws_access_key_id, aws_secret_access_key):
        self.ec2_client = boto3.client("ec2",
                                       region_name=region_name,
                                       aws_access_key_id=aws_access_key_id,
                                       aws_secret_access_key=aws_secret_access_key)
        self.ec2_resource = boto3.resource("ec2",
                                           region_name=region_name,
                                           aws_access_key_id=aws_access_key_id,
                                           aws_secret_access_key=aws_secret_access_key)

    def backup(self):
        reservations = self.ec2_client.describe_instances(
            Filters=[{'Name': 'tag-key', 'Values': ['backup']}])
        for reservation in reservations["Reservations"]:
            for instance in reservation["Instances"]:
                for tag in instance["Tags"]:
                    days = DEFAULT_KEEP_DAYS
                    if tag['Key'] == 'backup':
                        try:
                            days = int(tag['Value'])
                        except:
                            pass
                        instance_id = instance['InstanceId']
                        created_on = datetime.utcnow().strftime('%Y%m%d')
                        remove_on = (datetime.utcnow() +
                                     timedelta(days=days)).strftime('%Y%m%d')
                        name = f"InstanceId({instance_id})_CreatedOn({created_on})_RemoveOn({remove_on})"
                        try:
                            image = self.ec2_client.create_image(
                                InstanceId=instance_id, Name=name, NoReboot=NOREBOOT)
                            image_resource = self.ec2_resource.Image(
                                image['ImageId'])
                            image_resource.create_tags(Tags=[{'Key': 'RemoveOn', 'Value': remove_on},
                                                             {'Key': 'Name', 'Value': name}])
                        except:
                            print(f"Create AMI({name}) failed")
                        break

    def cleanup(self):
        today = datetime.utcnow().strftime('%Y%m%d')
        images = self.ec2_client.describe_images(
            Filters=[{'Name': 'tag:RemoveOn', 'Values': [today]}])
        for image_data in images['Images']:
            image = self.ec2_resource.Image(image_data['ImageId'])
            name_tag = [tag['Value']
                        for tag in image.tags if tag['Key'] == 'Name']
            if name_tag:
                print(f"Deregistering {name_tag[0]}")
            block_devices = copy.deepcopy(image.block_device_mappings)
            image.deregister()
            for dev in block_devices:
                if not 'Ebs' in dev:
                    continue
                snapshot_id = dev['Ebs']['SnapshotId']
                snapshot = self.ec2_resource.Snapshot(snapshot_id)
                snapshot.delete()


if __name__ == '__main__':
    bk = Backup(region_name=os.environ['REGION_NAME'],
                aws_access_key_id=os.environ['AWS_ACCESS_KEY_ID'],
                aws_secret_access_key=os.environ['AWS_SECRET_ACCESS_KEY'])
    # Backup EC2
    bk.backup()
    # Cleanup EC2
    bk.cleanup()
```
