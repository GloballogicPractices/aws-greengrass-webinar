## Introduction

This repo is a demo of Greengrass capabilities.
In this article you get a step-by-step guidance for setup of Greengrass Core, extracting the AWS infrastructure using CloudFormation, and deploying functions onto the Core.

Two kinds of setup are supported:
* Docker container
* Raspberry Pi

### IAM preparations
Prerequisites:
* Existing AWS CLI account (beware, costs may be charged)
* AWS CLI Installed

#### Create a Role, Policy, attach it to Greengrass Service
Create a service-linked role, remember the Role ARN in the output (used in the next steps).
```bash
aws iam create-role --role-name Greengrass_ServiceRole --assume-role-policy-document '{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "greengrass.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}'
```

Create a Role Policy. Use AWS-managed policy `AWSGreengrassResourceAccessRolePolicy`.
```bash
aws iam attach-role-policy --role-name Greengrass_ServiceRole --policy-arn arn:aws:iam::aws:policy/service-role/AWSGreengrassResourceAccessRolePolicy
```

Attach the role to the Greengrass Service in your account. Use Role ARN generated before.
```bash
aws greengrass associate-service-role-to-account --role-arn [ROLE_ARN]
```

#### Create a Core Device

Follow the [instructions in the AWS documentation](https://docs.aws.amazon.com/iot/latest/developerguide/iot-plant-step2.html "instructions in the AWS documentation"). Take the following into account:
* Create a thing (in this demo, it's desired to use names `my-raspberry-core-thing` or `my-docker-core-thing`).
* During creating, download the following files and store them in a secure place:
-- certificate,
-- public key,
-- private key.

* Remember the following values (you can find them later, see the hints in these documents):
-- Thing ARN (Details ~> Thing ARN)
-- Thing Certificate ARN (Security ~> Select the only certificate ~> Certificate ARN)
-- IoT Endpoint (Interact ~> HTTPS)

#### Create a Greengrass Policy and attach it to the Device
```bash
aws iot create-policy --policy-name my-greengrass-core-policy --policy-document '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": [
          "iot:Publish",
          "iot:Subscribe",
          "iot:Connect",
          "iot:Receive"
        ],
        "Resource": [
          "*"
        ]
      },
      {
        "Effect": "Allow",
        "Action": [
          "iot:GetThingShadow",
          "iot:UpdateThingShadow",
          "iot:DeleteThingShadow"
        ],
        "Resource": [
          "*"
        ]
      },
      {
        "Effect": "Allow",
        "Action": [
          "greengrass:*"
        ],
        "Resource": [
          "*"
        ]
      }
    ]
}'
```

Attach the policy to the Thing Certificate. Use the Certificate ARN you've got before.
```bash
aws iot attach-policy \
  --policy-name my-greengrass-core-policy \
  --target arn:aws:iot:us-west-2:531877574785:cert/63a8a448b56be6c2851ab3ea788a2e1e08a9a1d5a2edc331223731de35478c0c
```

### Configure the Core Device
In this sample, we show two setups:
* Docker
* Raspberry Pi.

Read the corresponding articles to start the Greengrass Cores.

### Exrtact the infrastructure (Core Group, Connectors, Lambdas) using CloudFormation

Use CloudFormation documentation.
The CloudFormation Templates can be found in this repo:
* `docker/gg-webinar-docker.yml` for Docker setup
* `rpi/gg-webinar-rpi.yml` for Raspberry Pi setup

### Deploy the Cloud Configurations to the Greengrass Core Device

Follow the instructions in [AWS docs](https://docs.aws.amazon.com/greengrass/latest/developerguide/configs-core.html).

Test the functionality with [AWS MQTT Client](https://docs.aws.amazon.com/iot/latest/developerguide/view-mqtt-messages.html).