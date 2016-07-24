**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-server>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-server/blob/master/modules/single-server/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Single Server Module

This module makes it easy to deploy a single server--that is, a single EC2 instance (e.g. a bastion host, Jenkins
server) rather than an Auto Scaling Group or ECS Cluster--along with the all the resources it typically needs:

1. The EC2 instance itself.
1. An Elastic IP (EIP) address.
1. An optional DNS record pointing at the EIP.
1. IAM Role and IAM instance profile.
1. Security group.

## How do you use this module?

* See the [root README](/README.md) for instructions on using Terraform modules.
* See the [examples](/examples) folder for example usage.
* See [vars.tf](./vars.tf) for all the variables you can set on this module.

Here is an example of how you might deploy a single Jenkins server with this module:

```hcl
module "jenkins" {
  source = "git::git@github.com:gruntwork-io/module-server.git//modules/single-server?ref=v0.0.40"

  name = "jenkins"
  ami = "ami-123456"
  instance_type = "t2.medium"
  keypair_name = "jenkins-keypair"
  user_data = "${var.user_data}"

  vpc_id = "${var.vpc_id}"
  subnet_id = "${var.subnet_id}"
}
```

If you already have a [Route 53 Hosted Zone](http://docs.aws.amazon.com/Route53/latest/DeveloperGuide/AboutHostedZones.html)
such as example.com, you could run this instance on jenkins.example.com by adding two more parameters:

```hcl
# The id of the example.com hosted zone
dns_zone_id = "ABC12345"

# The new DNS name to add
dns_name = "jenkins.example.com"
```

## How do I see the server?

This module includes several [Terraform outputs](https://www.terraform.io/intro/getting-started/outputs.html),
including:

1. `public_ip`: The public IP address of the server (via its EIP)
1. `fqdn`: The fully-qualified domain name of the server (e.g. jenkins.example.com) if you set the `dns_zone_id` and
   `dns_name` variables.

## How do I add custom security group rules?

One of the other important outputs of this module is the `security_group_id`, which is the id of the server's Security
Group. You can add custom rules to this Security Group using the `aws_security_group_rule` resource:

```hcl
module "jenkins" {
  source = "git::git@github.com:gruntwork-io/module-server.git//modules/single-server?ref=v0.0.40"

  # (... options omitted...)
}

# Custom rule to allow inbound HTTPS traffic from anywhere
resource "aws_security_group_rule" "allow_inbound_https_all" {
  type = "ingress"
  from_port = 443
  to_port = 443
  protocol = "tcp"
  cidr_blocks = ["0.0.0.0/0"]
  security_group_id = "${module.jenkins.security_group_id}"
}
```

## How do I add a custom IAM policy?

This module creates an IAM role for your EC2 instance and exports the id of that role as the output `iam_role_id`. You
can attach custom policies to this IAM role using the `aws_iam_policy_attachment` resource:

```hcl
module "jenkins" {
  source = "git::git@github.com:gruntwork-io/module-server.git//modules/single-server?ref=v0.0.40"

  # (... options omitted...)
}

resource "aws_iam_policy" "my_custom_policy" {
  name = "my-custom-policy"
  policy = " (... omitted ...) "
}

resource "aws_iam_policy_attachment" "attachment" {
  name = "example-attachment"
  roles = ["${module.jenkins.iam_role_id}"]
  policy_arn = "${aws_iam_policy.my_custom_policy.arn}"
}
```