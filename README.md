# week1-aws-s3-static

First Repo in FitHub



## \# Week1: S3 Static Site Demo



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

