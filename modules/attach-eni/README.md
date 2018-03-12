**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-server>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-server/blob/master/modules/attach-eni/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Attach ENI Scripts

This folder contains scripts you can use to attach [Elastic Network Interfaces 
(ENIs)](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html) to your EC2 Instances. 

1. `attach-eni`: This script can be run on an EC2 instance to attach an ENI. You can specify the ENI by ID or by having
   the script search for the ENI by tag. Normally, you would use Terraform's [aws_network_interface_attachment
   resource](https://www.terraform.io/docs/providers/aws/r/network_interface_attachment.html) to do this, but this
   resource does not work in cases where you need to attach volumes dynamically, such as with Auto Scaling Groups.

   The script has been tested with Ubuntu 16.04, AmazonLinux v1.x, and CentOS 7, though it may work on other Linux
   distributions as well.

An ENI allows you to have IP addresses that remain static, even if the underlying EC2 Instances are changing.

Check out the [attach-eni example](/examples/attach-eni) for how to use these scripts with Terraform.




## Installing the attach-eni script

You can install the `attach-eni` script using the [Gruntwork Installer](https://github.com/gruntwork-io/gruntwork-installer):

```bash
gruntwork-install --module-name "attach-eni" --repo "https://github.com/gruntwork-io/module-server" --tag "0.1.10"
```




## Using the attach-eni script

The `attach-eni` script has the following prerequisites:

1. It must be run as root
1. It must be run on an EC2 instance
1. The EC2 instance must have an IAM role with permissions to search ENIs and EC2 tags, as well as attach ENIs (see the
   [attach-eni example](/examples/attach-eni))
1. The EC2 instance must have the AWS CLI and jq installed

Typically, you'll want to run the `attach-eni` script in the User Data of your EC2 instances so it attaches the ENI at 
boot time.

Here is an example usage:

```bash
attach-eni --eni-id eni-abcd1234
```

This will attach ENI `eni-abcd1234` to the current Instance. Alternatively, you can also run the script as follows:

```bash
attach-eni --eni-with-same-tag Name
```

This tells the script to try find and attach an ENI with the same `Name` tag as the current EC2 Instance.
