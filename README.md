<!-- note marker start -->
**NOTE**: This repo contains only the documentation for the private BoltsOps Pro repo code.
Original file: https://github.com/boltopspro/enable-aws-guardduty/blob/master/README.md
The docs are publish so they are available for interested customers.
For access to the source code, you must be a paying BoltOps Pro subscriber.
If are interested, you can contact us at contact@boltops.com or https://www.boltops.com

<!-- note marker end -->

# Enable AWS GuardDuty CloudFormation Blueprint

[![BoltOps Badge](https://img.boltops.com/boltops/badges/boltops-badge.png)](https://www.boltops.com)

This blueprint enables AWS GuardDuty in your account. It can be used to enable GuardDuty on a single AWS account and single region. It can also be used to enable GuardDuty in multiple AWS accounts and regions. The same infrastructure code is used in both cases. This template is useful for compliance requirements.

StackSets are used with the [lono sets](https://lono.cloud/docs/stack-sets/lono-sets/) commands to deploy and enable GuardDuty with code.

Related Blueprints:

* [boltopspro/aws-config-aggregator](https://github.com/boltopspro-docs/aws-config-aggregator)
* [boltopspro/aws-config-bucket](https://github.com/boltopspro-docs/aws-config-bucket)
* [boltopspro/enable-aws-cloudtrail](https://github.com/boltopspro-docs/enable-aws-cloudtrail)
* [boltopspro/enable-aws-config](https://github.com/boltopspro-docs/enable-aws-config)
* [boltopspro/enable-aws-guardduty](https://github.com/boltopspro-docs/enable-guardduty)

## Usage

1. Add blueprint to Gemfile
2. Configure: configs/enable-aws-guardduty values
3. Deploy

## Add

Add the blueprint to your lono project's `Gemfile`.

```ruby
gem "enable-aws-guardduty", git: "git@github.com:boltopspro/enable-aws-guardduty.git"
```

## Configure

First you want to configure the `configs/enable-aws-guardduty` [config files](https://lono.cloud/docs/core/configs/).  You can use [lono seed](https://lono.cloud/reference/lono-seed/) to configure starter values quickly.

    lono seed enable-aws-guardduty

The generated files in `config/enable-aws-guardduty` folder look something like this:

    configs/enable-aws-guardduty/
    └── params
        └── development.txt

Here's an example params file.

configs/enable-aws-guardduty/

    # Parameter Group: AWS::GuardDuty::Detector
    # FindingPublishingFrequency=
    # MasterId=

## Deploy: Single Account and Region

Use the [lono cfn deploy](http://lono.cloud/reference/lono-cfn-deploy/) command to deploy. Example:

    lono cfn deploy enable-aws-guardduty

However, it is common to use StackSets to enable AWS Config in multiple regions and accounts.

## Deploy: Multiple Accounts and Regions

To deploy the stack to multiple accounts, we can use [lono sets](https://lono.cloud/docs/stack-sets/lono-sets/), which is essentially CloudFormation Stack Sets.

Configure the `configs/accounts` and `configs/regions` files (with your own values):

configs/accounts/development.txt

    111111111111
    222222222222

configs/regions/development.txt

    us-east-1
    us-west-2

Then you can deploy the template to multiple accounts and regions.

    lono sets deploy enable-aws-guardduty --sure # only creates the StackSet
    lono sets instances sync enable-aws-guardduty --sure # deploys StackSet instances - actual stacks

Deploying Stack Sets take a while as CloudFormation loops through each region one at a time. Note, using the "operation preferences" to increase the parallelism usually results in CloudFormation Rate Limit Errors and it failing. So, it is recommended running StackSets one stack at a time. Generally, StackSets are a nice feature and are helpful, but they can take a long time. Recommend using them only for simple templates.

The output looks something like this:

    $ lono sets instances sync enable-aws-guardduty --sure
    => Running create_stack_instances on:
      accounts: 111111111111,222222222222
      regions: us-east-1,us-west-2
    Stack Instance statuses... (takes a while)
    You can check on the StackSetsole Operations Tab for the operation status.
    Here is also the cli command to check:

        aws cloudformation describe-stack-set-operation --stack-set-name enable-aws-guardduty --operation-id 92089e7a-e9f5-49c2-bef1-0ad883cdd4c1

    2020-03-30 01:58:03PM Stack Instance: account 111111111111 region us-west-2 status OUTDATED reason User initiated operation
    2020-03-30 01:58:03PM Stack Instance: account 111111111111 region us-east-1 status OUTDATED reason User initiated operation
    2020-03-30 01:58:03PM Stack Instance: account 222222222222 region us-west-2 status OUTDATED reason User initiated operation
    2020-03-30 01:58:03PM Stack Instance: account 222222222222 region us-east-1 status OUTDATED reason User initiated operation
    Time took to complete stack set operation: 1m 20s
    Stack set operation completed.
    Stack Set Operation Summary:
    account 111111111111 region us-east-1 status SUCCEEDED
    account 222222222222 region us-east-1 status SUCCEEDED
    account 222222222222 region us-west-2 status SUCCEEDED
    account 111111111111 region us-west-2 status SUCCEEDED
    $

