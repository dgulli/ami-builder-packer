## Purpose

This Packer AMI Builder creates a new AMI out of the latest Amazon Windows 2016 AMI, and also provides a cloudformation template that leverages AWS CodePipeline to 
orchestrate the entire process.


## Source code structure

```bash
├── buildspec.yml                           <-- CodeBuild spec 
├── cloudformation                          
│   └── pipeline.yaml                       <-- Cloudformation to create entire pipeline
│   └── CodeBuild_VPC.yml                   <--- VPC for codebuild CFN template, if you dont alreayd have one.
├── packer_WinServer.json                   <-- Packer template for Pipeline
├── user_data.ps1                           <-- Powershell userdata
├── bootstrap.ps1                           <-- Powershell needed for WinRM bootstrap
```

## Cloudformation template

Cloudformation will create the following resources as part of the AMI Builder for Packer:

* ``cloudformation/pipeline.yaml``
    + GitHub - Git repository
    + AWS CodeBuild - Downloads Packer and run Packer to build AMI 
    + AWS CodePipeline - Orchestrates pipeline and listen for new commits in CodeCommit
    + Amazon SNS Topic - AMI Builds Notification via subscribed email
    + Amazon Cloudwatch Events Rule - Custom Event for AMI Builder that will trigger SNS upon AMI completion


## HOWTO

**Before you start**

* Install [GIT](https://git-scm.com/downloads) if you don't have it
* Make sure AWS CLI is configured properly
* [Configured AWS CLI and Git](http://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-https-unixes.html) to connect to AWS CodeCommit repositories

**Launch the Cloudformation stack**

Region | AMI Builder Launch Template
------------------------------------------------- | ---------------------------------------------------------------------------------
Sydney (ap-southeast-2) | [![Launch Stack](images/deploy-to-aws.png)](https://console.aws.amazon.com/cloudformation/home?region=ap-southeast-2#/stacks/new?stackName=Windows-AMI-Builder&templateURL=https://dg-windows-ami-builder.s3-ap-southeast-2.amazonaws.com/pipeline.yaml)


