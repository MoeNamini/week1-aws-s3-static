---
type: Page
title: README
description: null
icon: null
createdAt: '2025-09-22T17:12:36.793Z'
creationDate: 2025-09-22 20:42
modificationDate: 2025-09-22 22:57
tags: [4MonthPlan, CareerDevelopment, workflow]
coverImage: null
---

Of course. This version is designed to be a complete and detailed README for your GitHub repository. It incorporates all the files, commands, ARNs, and development notes you provided, structured to showcase your entire project lifecycle from a simple script to a fully automated and secure system.

---

# Secure AWS S3 Static Site with CI/CD & Automation

[https://github.com/MoeNamini/infra/actions/workflows/ci.yml](https://github.com/MoeNamini/infra/actions/workflows/ci.yml)

This repository documents the process of building a secure, automated infrastructure for hosting a static website on AWS S3. The project evolved from a basic manual setup to a robust solution using Infrastructure as Code (IaC), a full CI/CD pipeline with automated testing, and security best practices like least-privilege IAM policies, S3 bucket hardening, and secure access patterns.

## Table of Contents

### ## Key Workflows & Accomplishments

- **IAM Security:** Designed and implemented a least-privilege IAM policy, user, and group structure.

- **Infrastructure as Code (IaC):** Automated the creation of S3, IAM Policies, and IAM Roles using AWS CloudFormation.

- **Python Scripting:** Developed an importable S3 uploader function with a command-line interface using `boto3`.

- **Automated Testing:** Wrote unit tests using **Pytest** and the **Moto** library to mock AWS services for fast, local feedback.

- **Continuous Integration (CI/CD):** Configured a **GitHub Actions** workflow to run tests automatically on every `push` and `pull request`.

- **Local Development Automation:** Implemented a `pre-push` **Git hook** to run tests locally before code is pushed, ensuring no broken code reaches the repository.

- **Security Hardening:** Enabled Server-Side Encryption (SSE) and object versioning on the S3 bucket.

- **Secure Access:** Utilized S3 Pre-signed URLs for time-limited, secure access to private objects.

- **Policy Validation:** Used the AWS IAM Policy Simulator to verify permissions programmatically.

---

### ## Project Files Created

- **Application & Scripts:**

    - `script.py`: The core S3 uploader script. 

    - `create_policy.sh`: Helper script to create an IAM policy and return its ARN.

- **Infrastructure & Configuration (**`infra/`**):**

    - `s3-uploader-policy.json`: The least-privilege IAM policy document.

    - `leastprivilages3-policy.json`: An alternative version of the policy.

    - `trust-lambda.json`: The trust policy for the Lambda IAM Role.

    - `cloudformation/s3-and-iam.yml`: The CloudFormation template for provisioning all AWS resources.

- **Testing & CI/CD:**

    - `tests/test_s3_upload.py`: Pytest unit tests using the Moto library.

    - `tests/conftest.py`: Pytest configuration file.

    - `.github/workflows/ci.yml`: The GitHub Actions workflow definition.

    - `.git/hooks/pre-push`: Local Git hook to automate testing before pushes.

    - `.git/hooks/post-merge`: Local Git hook to automate testing after PR.

- **Artifacts & Documentation:**

    - `requirements.txt`: Python package dependencies.

    - `artifacts/reports.txt`: Append-only, human-readable test history.

    - `artifacts/pytest_full.txt`: Detailed test output for debugging.

    - `artifacts/junit.xml`: Test results formatted for CI/CD tool integration.

    <The scripts files are coded with AI **assistance**>

---

## Part 1: Initial Setup - Static Site & Uploader Script

The project began by creating a simple S3 bucket to host a static HTML file and a Python script to upload files to it.

- **Hosted Site URL:** [https://hybrid-moenamini02-static-01.s3.eu-central-1.amazonaws.com/index.html](https://hybrid-moenamini02-static-01.s3.eu-central-1.amazonaws.com/index.html)




#### S3 Uploader Script (`script.py`)

This `boto3` script provides a CLI to upload files to a specified S3 bucket.

```python
# script.py
import argparse
from pathlib import Path
import boto3
from typing import Optional, Union

def s3_client(region_name: Optional[str] = None, endpoint_url: Optional[str] = None):
    kwargs = {}
    if region_name:
        kwargs["region_name"] = region_name
    if endpoint_url:
        kwargs["endpoint_url"] = endpoint_url
    return boto3.client("s3", **kwargs)

def upload_file(path: Union[str, Path], bucket: str, key: str,
                region: Optional[str] = None, endpoint_url: Optional[str] = None) -> bool:
    """
    Uploads a local file to S3 using boto3 client.
    Returns True on success, raises exception on failure.
    """
    path = Path(path)
    if not path.exists():
        raise FileNotFoundError(f"{path} does not exist")

    client = s3_client(region_name=region, endpoint_url=endpoint_url)

    # Read bytes and put object (works for tests with moto)
    with path.open("rb") as f:
        client.put_object(Bucket=bucket, Key=key, Body=f.read())

    return True

def main():
    parser = argparse.ArgumentParser(description="Upload a file to S3")
    parser.add_argument("--file", required=True, help="Local path to file")
    parser.add_argument("--bucket", required=True, help="S3 bucket name")
    parser.add_argument("--key", required=True, help="S3 object key")
    parser.add_argument("--region", default=None, help="AWS region (optional)")
    parser.add_argument("--endpoint-url", default=None, help="S3 endpoint (for testing)")

    args = parser.parse_args()
    upload_file(args.file, args.bucket, args.key, region=args.region, endpoint_url=args.endpoint_url)

if __name__ == "__main__":
    main()
```

#### Example Usage & Output

```bash
py script.py --file "/path/to/your/index01.html" --bucket hybrid-moenamini02-static-01 --key "index01.html"

# Output:
# Uploaded /path/to/your/index01.html to s3://hybrid-moenamini02-static-01/index01.html
```

---

## Part 2: Security & IaC - IAM, Roles, and CloudFormation

The initial manual setup was evolved into a secure and automated system using IAM best practices and Infrastructure as Code.

### ## IAM Least-Privilege Configuration

A user, group, and policy were created to grant minimal necessary permissions for S3 operations.

- **Policy ARN:** `arn:aws:iam::<ACCOUNT_ID>:policy/HybridS3UploaderPolicy`

- **Group ARN:** `arn:aws:iam::<ACCOUNT_ID>:group/leastprivilage`

- **User ARN:** `arn:aws:iam::<ACCOUNT_ID>:user/fortest`

- **Role ARN:** `arn:aws:iam::<ACCOUNT_ID>:role/lambda-s3-LPP-role`

#### Implementation Commands

```bash
# Create the policy from a JSON file
aws iam create-policy --policy-name HybridS3UploaderPolicy --policy-document file://infra/s3-uploader-policy.json

# Create the group and attach the policy
aws iam create-group --group-name s3-uploaders
aws iam attach-group-policy \
  --group-name s3-uploaders \
  --policy-arn arn:aws:iam::<ACCOUNT_ID>:policy/HybridS3UploaderPolicy

# Create a user and add them to the group
aws iam create-user --user-name fortest
aws iam add-user-to-group --user-name fortest --group-name s3-uploaders
```

#### Created automated create_policy.sh scripts for better workflow and consistency

```bash
#!/usr/bin/env bash
set -e

# Use provided arguments or default values
POLICY_NAME=${1:-HybridS3UploaderPolicy}
POLICY_FILE=${2:-infra/new-policy.json}

# Create the policy and capture the ARN into a script variable
POLICY_ARN=$(aws iam create-policy --policy-name "$POLICY_NAME" --policy-document file://"$POLICY_FILE" \
  | jq -r '.Policy.Arn')

# Check if the ARN was captured successfully
if [[ -z "$POLICY_ARN" ]]; then
    echo "Error: Could not capture Policy ARN. Check previous command output." >&2
    exit 1
fi

# Print the result to the screen for confirmation
echo "✅ Policy ARN created: $POLICY_ARN"

# Append the ARN to the markdown file
echo "Policy ARN: $POLICY_ARN" >> iam-setup-template.md
```



### ##  Infrastructure as Code (IaC) with CloudFormation

The entire infrastructure (S3 bucket, IAM policy, IAM role) was codified in a CloudFormation template for repeatable, automated deployments.

#### CloudFormation Template (`infra/cloudformation/s3-and-iam.yml`)

```yaml
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
            Resource: !Sub arn:aws:s3:::${StaticBucket}/*

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
```

#### Deployment Command

```bash
aws cloudformation deploy \
  --template-file infra/cloudformation/s3-iam.yml \
  --stack-name hybrid-s3-iam \
  --capabilities CAPABILITY_IAM
```

---

##  Part 3: Automation & CI/CD

To ensure code quality and reliability, a complete testing and integration pipeline was established, automating validation both locally and in the cloud.

### ##  Local Testing Automation 

A `pre-push` hook was created to automatically run the Pytest suite before any code is pushed to the remote repository. This provides immediate feedback and prevents broken code from being committed. The script generates three distinct report artifacts:

- `reports.txt`: An append-only, human-readable log of test runs with timestamps.

- `pytest_full.txt`: A detailed report with full output for debugging failures.

- `junit.xml`: A standardized XML report for easy integration with CI/CD platforms like GitHub Actions.

    ```python
    # conftest.py
    import pytest
    import datetime
    from pathlib import Path
    
    REPORTS_DIR = Path("artifacts")
    REPORTS_FILE = REPORTS_DIR / "reports.txt"
    
    def pytest_runtest_logreport(report):
        """Hook to log results after each test phase."""
        if report.when == "call":  # only log test execution phase (not setup/teardown)
            REPORTS_DIR.mkdir(parents=True, exist_ok=True)
    
            test_name = report.nodeid  # includes test name + path
            test_dir = Path(report.fspath).parent
            timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    
            if report.passed:
                outcome = "PASS"
                message = ""
            else:
                outcome = "FAIL"
                message = str(report.longrepr)
    
            log_line = f"[{timestamp}] {test_name} | Dir: {test_dir} | {outcome} {message}\n"
    
            with open(REPORTS_FILE, "a", encoding="utf-8") as f:
                f.write(log_line)
    ```

This Pytest configuration file uses the pytest_runtest_logreport hook to automatically generate a custom, human-readable log entry in artifacts/reports.txt after every test run. This provides a clear and persistent audit trail of all test outcomes.

### ##  Continuous Integration with GitHub Actions (Git `pre-push` Hook)

The `.github/workflows/ci.yml` file configures a GitHub Actions workflow that automatically triggers the full test suite on every `push` and `pull_request` to the `main` branch, ensuring continuous validation.

```yaml
name: CI

# Triggers: run on pushes and pull requests to any branch
on:
  push:
    branches: ["**"]
  pull_request:
    branches: ["**"]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Python 3.11
        uses: actions/setup-python@v4
        with:
          python-version: "3.11"

      - name: Upgrade pip
        run: python -m pip install --upgrade pip

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run tests
        run: pytest -q

      # optional: upload test logs as artifact (uncomment if you want)
      - name: Upload test logs
        uses: actions/upload-artifact@v4
         with:
           name: pytest-report
           path: artifacts/reports.txt
```

###  ## Unit Testing Implementation Details

The core of the automation relies on Pytest and the Moto library to simulate AWS services for fast and reliable testing.

Unit Test for S3 Upload (infra/tests/test_s3_upload.py)
This test validates the upload_file function by using @mock_aws to create a virtual S3 environment. It confirms that a file can be successfully uploaded and that its contents are correct, all without touching the live AWS infrastructure.

```python
# infra/tests/test_s3_upload.py
import boto3
from moto import mock_aws
from pathlib import Path
import pytest

# import the function from your script (script.py must be at repo root)
from script import upload_file

@mock_aws
def test_upload_file(tmp_path):
    # choose region for client
    region = "eu-central-1"
    s3 = boto3.client("s3", region_name=region)

    bucket = "test-bucket"

    # create bucket; for non-us-east-1, include LocationConstraint
    if region == "us-east-1":
        s3.create_bucket(Bucket=bucket)
    else:
        s3.create_bucket(Bucket=bucket, CreateBucketConfiguration={"LocationConstraint": region})

    # prepare a sample file in a temporary directory (pytest tmp_path fixture)
    p = tmp_path / "test.txt"
    p.write_text("hello")

    # call the uploader function directly (in-process so moto intercepts boto3)
    assert upload_file(str(p), bucket=bucket, key="test.txt", region=region) is True

    # validate object exists and content matches
    obj = s3.get_object(Bucket=bucket, Key="test.txt")
    content = obj["Body"].read().decode("utf-8")
    assert content == "hello"
```

---

##  Part 4: Security Hardening & Validation

Security was further enhanced by validating policies and enabling protective features on the S3 bucket.

### ##  Policy Validation with IAM Simulator

The IAM Policy Simulator was used to programmatically verify that the IAM user had the correct permissions—and no more.

#### Simulation Command & Result

```bash
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::<ACCOUNT_ID>:user/dev-moe \
  --action-names s3:PutObject s3:GetObject s3:DeleteObject \
  --resource-arns arn:aws:s3:::hybrid-moenamini02-static-01/* \
  --profile admin01
```

```json
{
    "EvaluationResults": [
        {
            "EvalActionName": "s3:PutObject",
            "EvalResourceName": "arn:aws:s3:::hybrid-moenamini-static-01/*",
            "EvalDecision": "allowed",
            "MatchedStatements": [
                {
                    "SourcePolicyId": "leastprivilages3+delObj-policy",
                    "SourcePolicyType": "IAM Policy",
                    "StartPosition": { "Line": 9, "Column": 4 },
                    "EndPosition": { "Line": 19, "Column": 4 }
                }
            ]
        }
    ]
}
```

### ##  S3 Bucket Hardening

The S3 bucket was hardened by enabling Server-Side Encryption (SSE) and Versioning. This can be done through CLI or AWS console

```bash
# Enable Server-Side Encryption (SSE-S3) to protect data at rest
aws s3api put-bucket-encryption --bucket hybrid-moenamini02-static-01 --server-side-encryption-configuration '{"Rules":[{"ApplyServerSideEncryptionByDefault":{"SSEAlgorithm":"AES256"}}]}'

# Enable versioning to protect against accidental deletions and overwrites
aws s3api put-bucket-versioning --bucket hybrid-moenamini02-static-01 --versioning-configuration Status=Enabled
```

Also, CloudTrail can be activated for audit processes.

### ##  Secure Access with Pre-signed URLs

Generated a pre-signed URL to grant temporary, secure download access to a private object without exposing credentials.

```bash
aws s3 presign s3://hybrid-yourname-static-01/file-to-download.txt --expires-in 3600 --profile admin01
```

---

## Appendix: Project Development Notes

This section contains miscellaneous commands and notes from the development process.

### ## Git Workflow & Commands

- `git pull infra main --allow-unrelated-histories`: Pulled changes from a remote repository with a different history.

- `git push --set-upstream infra main`: Pushed local files to the new remote `infra` repository.

- `git config --global core.editor "notepad.exe"`: Set Notepad as the default Git editor on Windows.

### ## Git Practice & Learning

- `git reset` **(soft/hard/revert):** Practiced rewriting commit history.

- `git stash`**:** Used to save uncommitted changes in the working directory to switch branches.

- **Merge Conflicts:** Encountered and successfully resolved merge conflicts between branches.
