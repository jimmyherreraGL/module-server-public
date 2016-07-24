**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-server>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-server/blob/master/modules/route53-helpers/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Route 53 Helpers

This folder contains helper scripts for working with Amazon's DNS Service, [Route 53](https://aws.amazon.com/route53/).
The helpers are:

* `add-dns-a-record`: A script that can be run on an EC2 instance to add a DNS A record pointing to the instance's IP
  address.

Check out the [route53-helpers example](/examples/route53-helpers) for how to use these scripts with Terraform.

## Installing the helpers

You can install the helpers using the [Gruntwork Installer](https://github.com/gruntwork-io/gruntwork-installer):

```bash
gruntwork-install --module-name "route53-helpers" --repo "https://github.com/gruntwork-io/module-server" --tag "0.0.3"
```

## Using the add-dns-a-record helper

The `add-dns-a-record` script has the following prerequisites:

1. It must be run on an EC2 instance
1. The EC2 instance must have an IAM role with permissions to modify Route 53 entries (see below)
1. The EC2 instance must have the AWS CLI and jq installed

To run the script, simply pass it the AWS region, the id of the [hosted 
zone](http://docs.aws.amazon.com/Route53/latest/DeveloperGuide/AboutHostedZones.html) where the DNS entry should be 
added, and the hostname to use for the DNS entry:

```bash
add-dns-a-record --aws-region us-east-1 --hosted-zone-id ASDF123456 --hostname foo.my-company.com
```

Run `add-dns-a-record --help` to see all available options.

## IAM Permissions

Here is an example of an IAM policy your EC2 instance needs attached to its IAM role to run these scripts:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Action": "route53:ChangeResourceRecordSets",
      "Effect": "Allow",
      "Resource": "arn:aws:route53:::hostedzone/<ID_OF_HOSTED_ZONE>"
    }
  ]
}
```

Check out the [route53-helpers example](/examples/route53-helpers) to see what this looks like in action.