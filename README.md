# week1-aws-s3-static

First Repo in FitHub



### \## What I built



#### Day1:

Static site hosted on S3, plus a Python uploader script.

https://hybrid-moenamini02-static-01.s3.eu-central-1.amazonaws.com/index.html



\## How to run locally

Commands used:

\# script.py — simple S3 uploader using boto3



""

import os

import argparse

import boto3

from botocore.exceptions import ClientError



def upload\_file(file\_path, bucket, object\_name=None):

    if object\_name is None:

        object\_name = os.path.basename(file\_path)

    s3\_client = boto3.client('s3')

    try:

        s3\_client.upload\_file(file\_path, bucket, object\_name)

        print(f"Uploaded {file\_path} to s3://{bucket}/{object\_name}")

    except ClientError as e:

        print(f"Error: {e}")

        return False

    return True



if \_\_name\_\_ == "\_\_main\_\_":

    parser = argparse.ArgumentParser(description="Upload file to S3")

    parser.add\_argument("--file", required=True, help="Path to file to upload")

    parser.add\_argument("--bucket", required=True, help="S3 bucket name")

    parser.add\_argument("--key", required=False, help="S3 object key (optional)")

    args = parser.parse\_args()

    upload\_file(args.file, args.bucket, args.key)

""



""

(.venv) PS C:\\Users\\----\\PycharmProjects\\PythonProject1\\.venv> py script.py --file "C:\\Users\\moham\\OneDrive\\Desktop\\GitHub\\week1-aws-s3-static\\index01.html" --bucket hybrid-moenamini02-static-01 --key "index01.html"

Uploaded C:\\Users\\----\\--------\\Desktop\\GitHub\\week1-aws-s3-static\\index01.html to s3://hybrid-moenamini02-static-01/index01.html

""





\## AWS resources created

\- S3 bucket: hybrid-moenamini02-static-01

\- IAM user: dev-moenamini (programmatic)



\## Notes

\- Be cautious of public access and billing.



#### Day2:



create .infra/

infra/

├── s3-uploader-policy.json              # least-privilege policy (object + list)

├── trust-lambda.json                    # trust policy for Lambda role

├── create\_policy.sh                     # helper script to create policy \& return ARN

├── cloudformation/

│   └── s3-and-iam.yml                   # CloudFormation template to create S3 + policy + role

├── iam-setup-template.md                # filled template you should update with outputs

├── tests/

│   └── test\_s3\_upload.py                # pytest using moto to validate uploader script

└── workflows/

    └── ci.yml                           # GitHub Actions workflow to run tests

Created the "infra" repo in GitHub.

-pulled the changes from GitHub repo: git pull infra main --allow-unrelated-histories
-pushed the files from local repo: git push --set-upstream infra main



Set notepad as global text editor instead of vim: git config --global core.editor "notepad.exe"

group leastprivilage and user fortest created, policy attached to group
created iam user access ket, set it as local profile, delete the old access key



Created a s3 policy uploader script which create the policy saved as a .json file in AWS and save the policy's ARN into a template that include all ARNs, with one CLI command "./create\_policy.sh MyNewPolicy infra/new-policy.json"

this is test01
testing merge conflict 


create role and attach policy 
aws iam create-role --role-name lambda-s3-LLP-role --assume-role-policy-document file://infra/trust-lambda.json



aws iam attach-role-policy --role-name lambda-s3-LLP-role --policy-arn arn:aws:iam::\*\*\*\*\*\*\*\*



aws iam attach-role-policy --role-name lambda-s3-LLP-role --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole



created a couldformation for automated lambda role/s3 bucket creation

AWSTemplateFormatVersion: '2010-09-09'

Description: S3 bucket + least-privilege policy + lambda role



Resources:

 StaticBucket:

   Type: AWS::S3::Bucket

   Properties:

     BucketName: hybrid-yourname-static-01

     WebsiteConfiguration:

       IndexDocument: index.html



 S3UploaderPolicy:

   Type: AWS::IAM::ManagedPolicy

   Properties:

     # This line is removed. CloudFormation will generate a name.

     PolicyDocument:

       Version: '2012-10-17'

       Statement:

         - Sid: AllowListBucket

           Effect: Allow

           Action: s3:ListBucket

           Resource: !Sub arn:aws:s3:::${StaticBucket}

         - Sid: AllowObjectActions

           Effect: Allow

           Action:

             - s3:GetObject

             - s3:PutObject

           Resource: !Sub arn:aws:s3:::${StaticBucket}/\*



 LambdaAssumeRole:

   Type: AWS::IAM::Role

   Properties:

     # This line is removed. CloudFormation will generate a name.

     AssumeRolePolicyDocument:

       Version: "2012-10-17"

       Statement:

         - Effect: Allow

           Principal: { Service: lambda.amazonaws.com }

           Action: sts:AssumeRole

     ManagedPolicyArns:

       - !Ref S3UploaderPolicy

       - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole



Created new branches 

this is test02
testing merge conflict
This line is added in VS Code editor 

Practicing git reset-soft-hard
This is line 1
This is line 2 
This is line 3