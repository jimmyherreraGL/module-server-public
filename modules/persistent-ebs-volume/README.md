**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-server>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-server/blob/master/modules/persistent-ebs-volume/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Persistent EBS Volume Scripts

This folder contains scripts you can use to work with EBS Volumes that persist between redeploys of an EC2 instance:

1. `mount-ebs-volume`: This script can be run on an EC2 instance to attach and mount a persistent EBS volume. It uses
   the AWS API to attach the volume, creates a file system on it (if it doesn't have one already), and mounts it at a
   specified mount point. If you're mounting volumes dynamically, such as with instances in an Auto Scaling Group, you 
   can pass in multiple volume ids and this script will try to attach each one until it finds one in the same 
   availability zone that is unattached.
1. `unmount-ebs-volume`: This script can be run on an EC2 instance to unmount and detach a persistent EBS volume. It
   unmounts the volume at the specified mount point and then uses the AWS API to detach the volume.

Why not use the `aws_ebs_volume` and `aws_volume_attachment` resources in Terraform to do this for us? Because:

1. [Due to a bug](https://github.com/hashicorp/terraform/issues/2957#issuecomment-150613677), the 
   `aws_volume_attachment` resource does not work correctly and cannot currently be used.
1. The `aws_volume_attachment` resource does not work for dynamically attaching volumes, such as with an Auto Scaling
   Group.

A persistent EBS volume is useful for servers that persist data to the local hard disk and need that data to still be
available after a redeploy, such as a database like MySQL, CI server like Jenkins, or CMS like WordPress. Note, an EBS
volume can only be associated with a single EC2 Instance, so if you need the data on the disk to be shared amongst
multiple servers, check out the [Amazon Elastic File System](https://aws.amazon.com/efs/), which provides a service
built on top of NFS.

Check out the [persistent-ebs-volume example](/examples/persistent-ebs-volume) for how to use these scripts with
Terraform.

## Installing the scripts

You can install these scripts using the [Gruntwork Installer](https://github.com/gruntwork-io/gruntwork-installer):

```bash
gruntwork-install --module-name "persistent-ebs-volume" --repo "https://github.com/gruntwork-io/module-server" --tag "0.0.3"
```

## Using the scripts

The scripts have the following prerequisites:

1. They must be run as root
1. They must be run on an EC2 instance
1. The EC2 instance must have an IAM role with permissions to list, attach, and detach volumes (see below)
1. The EC2 instance must have the AWS CLI and jq installed

Run the `mount-ebs-volume` script in the User Data of your EC2 instances so it mounts the volume at boot. Run the 
`unmount-ebs-volume` script just before your instance shuts down (make sure nothing is writing to the volume when
running this script).

Here is an example usage:

```bash
mount-ebs-volume --aws-region us-east-1 --volume-id vol-123456 --device-name /dev/xvdf --mount-point /data --owner ubuntu
```

This will attach volume `vol-123456` as device `/dev/xvdf` and then mount it at the path `/data` (see [Device Naming on
Linux Instances](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/device_naming.html) for an explanation of why we
chose `/dev/xvdf`). For dynamic setups where you don't have a static mapping of instances to volumes, such as in an
Auto Scaling Group, you can specify the `--volume-id` parameter multiple times:

```bash
mount-ebs-volume --aws-region us-east-1 --volume-id vol-123456 --volume-id vol-7891011 --volume-id vol-12131415 --device-name /dev/xvdf --mount-point /data --owner ubuntu
```

This tells the script to try each volume until it finds one that it can attach. Run `mount-ebs-volume --help` to see 
all the options.

## IAM Permissions

Here is an example of an IAM policy your EC2 instance needs attached to its IAM role to run these scripts:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Action": [
        "ec2:AttachVolume",
        "ec2:DetachVolume"
      ],
      "Effect": "Allow",
      "Resource": [
        "arn:aws:ec2:us-east-1:<YOUR_AWS_ACCOUNT_ID>:volume/<ID_OF_EBS_VOLUME>",
        "arn:aws:ec2:us-east-1:<YOUR_AWS_ACCOUNT_ID>:instance/<ID_OF_EC2_INSTANCE>"
      ]
    },
    {
      "Sid": "",
      "Action": "ec2:DescribeVolumes",
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
```

Check out the [persistent-ebs-volume example](/examples/persistent-ebs-volume) to see what this looks like in action.