Multi-Region CI/CD Pipeline with AWS

Overview

This project sets up a three-stage CI/CD pipeline that deploys an application across multiple AWS Regions using the following services:

AWS CodeCommit – Source code management

AWS CodeBuild – Builds and tests the application

AWS CloudFormation – Provisions and manages infrastructure

AWS CodePipeline – Orchestrates the entire pipeline

Architecture

The pipeline consists of three main stages:

Source Stage – Code is pushed to AWS CodeCommit

Build Stage – AWS CodeBuild compiles the code and runs tests

Deploy Stage – AWS CloudFormation deploys infrastructure across multiple regions

Additional services used:

Amazon S3 – Artifact storage

Amazon SNS – Notification system

IAM – Role and permission management

EC2 – Local testing environment

Table of Contents

Overview

Architecture

Prerequisites

Setup Instructions

Usage

Contributing

License

Prerequisites

An AWS account with admin permissions

AWS CLI configured on your local machine

Basic understanding of CodeCommit, CodeBuild, CodePipeline, and CloudFormation

Application code to deploy

IAM Roles/Policies:

EC2FullAccess

SSM AutomationRole

IAMFullAccess

Create an SNS topic for notifications:

Name: Code_Ready_4_Prod

Add email subscription to receive approval notifications

Setup Instructions

Step 1: Create a Local Testing Environment

Launch an EC2 instance in us-east-1 named local-test-env

Attach an admin IAM role to the instance

Install tools:

python3 --version  # Should return version info
sudo yum update -y
sudo yum install -y python3 python3-pip
pip3 install cfn-lint

Upload CloudFormation template to S3:

aws s3 mb s3://3stages-cicd-bucket
aws s3 cp Ec2_cfn_template.yml s3://3stages-cicd-bucket/

Copy template to EC2 and validate:

aws s3 cp s3://3stages-cicd-bucket/Ec2_cfn_template.yml .
cfn-lint Ec2_cfn_template.yml

Package and deploy:

aws cloudformation package \
  --template-file Ec2_cfn_template.yml \
  --s3-bucket 3stages-cicd-bucket \
  --output-template-file build_template_Artifact.yml

Then deploy using CloudFormation Console.

Step 2: Set Up CodeCommit

Go to CodeCommit Console → Create repo: 3stage-cicd-source-repo

Clone repo locally:

git clone https://git-codecommit.us-east-1.amazonaws.com/v1/repos/3stage-cicd-source-repo
cd 3stage-cicd-source-repo

Step 3: Configure CodeBuild

Go to CodeBuild Console → Create new project:

Project Name: 3stage-cicd-Buildproject

Source: CodeCommit (3stage-cicd-source-repo)

Environment: Amazon Linux 2023, aws/codebuild/amazonlinux2-x86_64-standard:3.0

Buildspec file: buildspec.yml

Sample buildspec.yml:

version: 0.2
phases:
  install:
    commands:
      - echo Installing dependencies
  build:
    commands:
      - echo Build started
      - echo Build completed
artifacts:
  files:
    - '**/*'

Step 4: Create CloudFormation Templates

Design templates for both test and production environments

Parameterize regions, instance types, and networking

Step 5: Configure CodePipeline

Go to CodePipeline Console → Create pipeline 3stage-cicd-pipeline

Source Stage: CodeCommit → 3stage-cicd-source-repo (master branch)

Build Stage: AWS CodeBuild → 3stage-cicd-Buildproject

Deploy Stage:

Deploy to Test (us-east-1)

Action: DeployToTest

Region: us-east-1

Stack Name: cfn-test-env-stack

Template: build_template_Artifact.yml

Role: CloudFormationServiceRoleForMyTestPipeline

Manual Approval

SNS Topic: Code_Ready_4_Prod

Reviewer emails: Add to the subscription list

Deploy to Production (us-west-1)

Action: DeployToProd

Region: us-west-1

Stack Name: cfn-prod-env-stack

Template: build_template_Artifact.yml

Role: CloudFormationServiceRoleForMyTestPipeline

Parameter overrides:

{
  "KeyName": "cfn_kp-us-west-1",
  "VPCStackName": "SampleNetworkCrossStack"
}

Usage

Commit code to the 3stage-cicd-source-repo

CodePipeline will automatically build and deploy to test and production (with manual approval)

Contributing

Contributions are welcome! Please fork the repository and use a feature branch. Pull requests are accepted.

License

This project is licensed under the MIT License – see the LICENSE file for details.

About

This project demonstrates a real-world, production-ready CI/CD pipeline built with AWS native services, ideal for DevOps and infrastructure automation learning or showcase.
