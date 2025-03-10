---
title: "Level 300: Optimization Data Collection"
#menutitle: "Lab #1"
date: 2020-10-22T11:16:08-04:00
chapter: false
weight: 8
hidden: false
---
## Last Updated
October 2021

## Authors
- Stephanie Gooch, Commercial Architect (AWS)
- Yuriy Prykhodko, Sr. Technical Account Manager (AWS)
- Iakov Gan, Sr. Technical Account Manager (AWS)


## Feedback
If you wish to provide feedback on this lab, report an error, or you have a suggestion, please email: costoptimization@amazon.com


## Introduction
Amazon Web Services offers a broad set of global cloud-based products including compute, storage, databases, analytics, networking, mobile, developer tools, management tools, security and enterprise applications. These services help organizations move faster, lower IT costs and scale.

This lab is designed to **enable you to collect utilization data from different AWS services to help you identify optimization opportunities**. This lab provides set of optional modules to automate data collection and explains how to create custom modules to pull additional data sets. 

The main sources of the data used in optional modules:
* [Cost Explorer Rightsizing Recommendations](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/ce-rightsizing.html)
* [AWS Trusted Advisor](https://aws.amazon.com/premiumsupport/technology/trusted-advisor/)
* [AWS Compute Optimizer](https://aws.amazon.com/compute-optimizer/)
* [Amazon EC2](https://aws.amazon.com/ec2/?ec2-whats-new.sort-by=item.additionalFields.postDateTime&ec2-whats-new.sort-order=desc) service inventories like Amazon EBS volumes, snapshots and AMIs
* [Amazon Elastic Container Service](https://aws.amazon.com/ecs/) chargeback data 
* [Amazon Relational Database Service](https://aws.amazon.com/rds/) utilization data
* [AWS Organizations](https://aws.amazon.com/organizations/) data export 
* [AWS Budgets Export](https://aws.amazon.com/aws-cost-management/aws-budgets/) data export

## Architecture 
Resources for this lab deployed with AWS CloudFormation:
1. **Optimization Data Collection** Stack deploys core resources for the lab and allows to choose which [data collection modules](../300_optimization_data_collection/3_data_collection_modules) to deploy. Each data collection module is optional. We recommend to deploy this stack in separate optimization data collection AWS account. 
1. **Optimization Management Data Role** Stack deploys AWS IAM Role for AWS Lambda which allows read-only access to retrieve linked accounts information from AWS Organizations. This stack should be deployed in management AWS account
1. **Optimization Data Collection** StackSet deploys IAM role required for AWS Lambda to get optimization data for each module. StackSet should be deployed from either organization's management account or a delegated administrator account to all linked accounts in organization. 

![Images/Arc.png](/Cost/300_Optimization_Data_Collection/Images/Arc.png)

Resources deployed with Optimization Data Collection Stack launch following workflow:

**Collecting linked account information**: 
1. [Amazon EventBridge](https://aws.amazon.com/eventbridge/) rule invokes account collector [AWS Lambda](https://aws.amazon.com/lambda/) based on schedule in optimization data collection account. By default schedule triggers Lambda function every 14 days and can be adjusted if needed.
2. The Lambda function assumes [AWS Identity and Access Management](https://aws.amazon.com/iam/) (IAM) role in management account, retrieves linked accounts ids and names via [AWS Organizations](https://aws.amazon.com/organizations/) SDK and sends them to [Amazon Simple Queue Service](https://aws.amazon.com/sqs/) (SQS) queues for every deployed data collection module.  

**Collecting optimization data from linked accounts**:

3. Messages in SQS queue trigger Lambda functions for each data collection module
4. Each data collection module Lambda function assumes IAM role in linked accounts listed in SQS messages and retrieves respective optimization data via [AWS SDK for Python](https://aws.amazon.com/sdk-for-python/). Collected data stored in data collection [Amazon S3](https://aws.amazon.com/s3/) bucket

**Analyzing and visualizing optimization data**:

5. Once data stored in S3 bucket, Lambda function triggers [AWS Glue](https://aws.amazon.com/glue/) crawler which creates or updates table in Glue Data Catalog
6. Optimization data can be queried and analyzed with [Amazon Athena](https://aws.amazon.com/athena) or visualized with [Amazon QuickSight](https://aws.amazon.com/quicksight/) to get optimization recommendations 

It is possible to deploy **Optimization Data Collection** Stack to organization's management account. Deployment in such case will look like on architecture diagram below
{{%expand "Architecture for management account deployment" %}}
![Images/Arc_mngmt_acct.png](/Cost/300_Optimization_Data_Collection/Images/Arc_mngmt_acct.png)
{{% /expand%}}

## Goals
- Deploy core resources and data collection modules
- Collect optimization data in S3 bucket
- Query and analyze optimization data with Amazon Athena or visualize it with Amazon QuickSight


## Prerequisites
- Access to the Management AWS Account of the AWS Organization to deploy Cloudformation
- Access to a sub account within the Organization - referred to as **Cost Optimization Account**
- Completed the [Account Setup Lab]({{< ref "/Cost/100_Labs/100_1_AWS_Account_Setup" >}})
- Completed the [Cost and Usage Analysis lab]({{< ref "/Cost/200_Labs/200_4_Cost_and_Usage_Analysis" >}})
- Completed the [Cost Visualization Lab]({{< ref "/Cost/200_Labs/200_5_Cost_Visualization" >}}) 

## Deployment Options
We suggest you do not deploy the main resources and collectors into your management account and instead use the cost account created in [Account Setup Lab]({{< ref "/Cost/100_Labs/100_1_AWS_Account_Setup" >}}). However, Some IAM resources will be needed to read data from the management account. 

## Permissions required

Be able to create the below in the management account:
- IAM role and policy
- Deploy CloudFormation

Be able to create the below in a sub account where your CUR data is accessible:
- Deploy CloudFormation
- Amazon S3 Bucket 
- AWS Lambda function 
- IAM role and policy
- Amazon CloudWatch trigger
- Amazon Athena Table
- AWS Glue Crawler


## Costs
- Estimated costs should be <$5 a month for small Organization 


## Time to complete
- 30 minutes

## Steps:
{{% children  /%}}


{{< prev_next_button link_next_url="./1_deploy_main_resources/" button_next_text="Start Lab" first_step="true" />}}
