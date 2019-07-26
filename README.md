## Purpose

This Packer AMI Builder creates a new AMI out of the latest Amazon Windows 2016 AMI, and also provides a cloudformation template that leverages AWS CodePipeline to 
orchestrate the entire process.


## Source code structure

```bash
├── buildspec.yml                           <-- CodeBuild spec 
├── CodeBuild_VPC.yml                       <--- VPC for codebuild CFN template, if you dont alreayd have one.
├── cloudformation                          <-- Cloudformation to create entire pipeline
│   └── pipeline.yaml
├── packer_WinServer.json                         <-- Packer template for Pipeline
├── user_data.ps1                        <-- Powershell userdata
├── bootstrap.ps1                       <-- Powershell needed for WinRM bootstrap
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

**To clone the AWS CodeCommit repository (console)**

1.  From the AWS Management Console, open the AWS CloudFormation console.
2.  Choose the AMI-Builder-Blogpost stack, and then choose Output.
3.  Make a note of the Git repository URL.
4.  Use git to clone the repository.
For example: git clone https://github.com/dgulli/ami-builder-packer.git

**To clone the AWS CodeCommit repository (CLI)**

```bash
# Retrieve CodeCommit repo URL
git_repo=$(aws cloudformation describe-stacks --query 'Stacks[0].Outputs[?OutputKey==`GitRepository`].OutputValue' --output text --stack-name "AMI-Builder-Blogpost")

# Clone repository locally
git clone ${git_repo}
```

Next, we need to copy all files in this repository into the newly cloned Git repository:

* Download [ami-builder-packer ZIP](https://github.com/awslabs/ami-builder-packer/archive/master.zip).
* Extract and copy the contents to the Git repo

Lastly, commit these changes to your AWS CodeCommit repo and watch the AMI being built through the AWS CodePipeline Console:

```bash
git add .
git commit -m "SHIP THIS AMI"
git push origin master
```

![AWS CodePipeline Console - AMI Builder Pipeline](images/ami-builder-pipeline.png)


