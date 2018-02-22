**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-server>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-server/blob/master/examples/attach-eni/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Attach ENI Example

This folder contains an example of how to create EC2 instances with [Elastic Network Interfaces 
(ENIs)](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html) attached. This is useful for servers 
that need IP addresses that remain static even if the underlying EC2 Instances change (e.g., ZooKeeper). Ideally, you
would attach these ENIs using Terraform's [aws_network_interface_attachment
resource](https://www.terraform.io/docs/providers/aws/r/network_interface_attachment.html), but it does not work with
dynamic use cases, such as Auto Scaling Groups. 

Therefore, this example includes a Packer template that installs the [attach-eni script](/modules/attach-eni) and uses 
it to attach an ENI when the Instances are booting. One of the examples shows how to attach an ENI by ID and the other
by tag.




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