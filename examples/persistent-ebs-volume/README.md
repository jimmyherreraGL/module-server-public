**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-server>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-server/blob/master/examples/persistent-ebs-volume/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Persistent EBS Volume Example

This folder contains an example of how to create an EC2 instance with two EBS volumes that are persisted between
redeploys. This is useful for servers that persist data to the local hard disk and need that data to still be available
after a redeploy, such as a database like MySQL, CI server like Jenkins, or CMS like WordPress. Although it should be
possible to do this with native Terraform using the `aws_ebs_volume` and `aws_volume_attachment` resources, [due to a
bug](https://github.com/hashicorp/terraform/issues/2957#issuecomment-150613677), the `aws_volume_attachment` resource
does not work correctly and cannot be used. Therefore, this example includes a Packer template that installs the
[persistent-ebs-volume scripts](/modules/persistent-ebs-volume) and uses those scripts to attach and mount the volume 
when the instance is booting.

Note, an EBS volume can only be associated with a single EC2 Instance, so if you need the data on the disk to be shared
amongst multiple servers, check out the [Amazon Elastic File System](https://aws.amazon.com/efs/), which provides a
service built on top of NFS.

## How do you run this example?

To run this example, you need to do the following:

1. Build an AMI using Packer
1. Deploy the AMI using Terraform

These steps are described in detail next.

### Build an AMI using Packer

The code that runs the EC2 instance in this example is an [Amazon Machine Image
(AMI)](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) that has been defined in a [Packer
template](https://www.packer.io/) under `packer/build.json`. To build an AMI from this template:

1. Install [Packer](https://www.packer.io/).
1. Set up your [AWS credentials as environment variables](https://www.packer.io/docs/builders/amazon.html).
1. Run `packer build build.json` to create the AMI in your AWS account. Note down the ID of this new AMI.

### Deploy the AMI using Terraform

Now that you have an AMI, use Terraform to deploy it:

1. Install [Terraform](https://www.terraform.io/).
1. Open up `vars.tf` and set secrets at the top of the file as environment variables and fill in any other variables in
   the file that don't have defaults. This includes the `ami` variable which you should fill in with the ID of the
   AMI you just built with Packer.
1. `terraform get`.
1. `terraform plan`.
1. If the plan looks good, run `terraform apply`.

When the templates are applied, Terraform will output the IP address of the EC2 instance. If you SSH to the instance,
it should have a `/data` folder. This is the EBS volume. It should persist whatever data you put into it in between
restarts of this EC2 instance.