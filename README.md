# Debugging serverless applications

*Note: The following workshop series is customized for AWS re:Invent 2019 Builders Session on **"DOP340: Debugging serverless applications"** and may contain content that is based on having resources provided during the workshop.*

In this workshop, you'll learn how to better manage Lambda Applications, use [Serverless Application Model (SAM)](https://github.com/awslabs/serverless-application-model) to debug locally and try out X-Ray to trace code errors & performance issues.

You will use SAM to deploy API interfaces, business logic, and database into your AWS account, based on the [Wild Rydes Serverless Workshops](https://github.com/aws-samples/aws-serverless-workshops). The RESTful API will allow a user to list, create, view, update, and delete the unicorns in the Wild Rydes stable.

The application architecture uses [AWS Lambda](https://aws.amazon.com/lambda/), [Amazon API Gateway](https://aws.amazon.com/api-gateway/), and [Amazon DynamoDB](https://aws.amazon.com/dynamodb/).  The API is built using Lambda and API Gateway, using DynamoDB as a persistent data store for unicorn data.

See the diagram below for a depiction of the API architecture.

![Wild Rydes DevOps RESTful API Application Architecture](images/wildrydes-devops-api-architecture.png)

The serverless application that you will deploy runs on a DevOps Continuous Delivery Pipeline which uses [AWS CodePipeline](https://aws.amazon.com/codepipeline/), [AWS CodeBuild](https://aws.amazon.com/codebuild/), and [Amazon S3](https://aws.amazon.com/s3/).  CodePipeline orchestrates the steps to build, test, and deploy your code changes.  CodeBuild compiles source code, runs tests, and produces software packages that are ready to deploy to environments.

<!--
See the screenshot below for a depiction of the continuous delivery pipeline that you will build at the completion of Module 4.

![Wild Rydes Unicorn API Continuous Delivery Pipeline](images/codepipeline-final.png)
-->

## Prerequisites

### 1. re:Invent 2019 Builders Session Lab Account

For this workshop, we will be issuing lab accounts for you to use to try out the services as documented. It is just like any other AWS account (with some IAM restrictions) and is completely **free to use**. You will not be charged for running any resources in these accounts. Do not store any sensitive information on these accounts. Note that the accounts will be terminated after the workshop, so do take measures to backup and save any work done.

> **Note:** As AWS accounts are a finite resource and we have obtained approval to provision them solely for this workshop's purpose, we'd like to ask you to be **frugal** on the usage.

1. Prior to the workshop, you will receive a **12-character team hash**. If you do not have this, please sound out to the workshop facilitator for assistance.

2. Head over to the [Event Engine Dashboard](https://dashboard.eventengine.run/) to login with the hash.

  ![Hash](images/hash_login.png)

3. Once you have logged in, you will be greeted with the **User Dashboard**. The AWS console can be access by clicking on the **AWS Console** button on the right.

  ![DevOps Workshop Architecture](images/console.png)
  ![DevOps Workshop Architecture](images/console2.png)

> **Important:** Please use the region **US West - N.California (us-west-1)** for all resources in this lab.

### 2. AWS credentials

In this workshop, you will be interacting with AWS resources from your local workstation, such as pulling/pushing code into your CodeCommit repository, run CloudFormation templates etc. As such, you will need to run these actions with an AWS credentials that has the permissions to do so. We will use the AWS Console to setup an IAM user with **Administrator** rights so that your local workstation can assume this role.

> Note: We are using an Administrator role in this workshop to simplify the steps required in and is NOT recommended in a Production environment.

<details>
<summary><strong>HOW TO create IAM user with Administrator permission (expand for details)</strong></summary><p>

**Creating IAM users from the AWS console**

1. Sign in to the AWS Management Console and open the IAM console at [https://console.aws.amazon.com/iam/](https://console.aws.amazon.com/iam/).

2. In the navigation pane, choose **Users** and then choose **Add user**.

3. Type the user name for the new user. This is the sign-in name for AWS.

4. Select the type of access this set of users will have. Select both **Programmatic Access** and **AWS Management Console**
   + **Programmatic access** are for users that require access to the API, AWS CLI, or Tools for Windows PowerShell. This creates an access key for each new user. Download the key pair by clicking on **Download .csv file**. Store the keys in a secure location.
   + **AWS Management Console access** are for users that require access to the AWS Management Console. Users use a username and password to access the console. We will use this access to view and verify resources created.

5. Choose **Next: Permissions**.

6. On the **Set permissions** page, specify how you want to assign permissions to this set of new users. Choose **Attach existing policies to user directly**.

7. In the search box, type in **Administrator**. Select the Administrator policy to attach to the user.

</p></details>
<p>

### 3. AWS Toolkit for Visual Studio Code

Throughout this workshop, we will be assuming that you are using [Visual Studio Code](https://code.visualstudio.com/) with AWS Toolkit installed. You may use other IDEs of your choice but the screens and steps would differ.

<details>
<summary><strong>HOW TO install AWS Toolkit for VS Code (expand for details)</strong></summary><p>

Before you can install the Toolkit for VS Code, you must have the following:
- VS Code version 1.31.1 or later [VS Code download](https://code.visualstudio.com/) page.
- Node.js SDK: https://nodejs.org/en/download

#### Installing AWS Toolkit for VS Code:
1. Start the VS Code editor.

2. In the Activity Bar on the side of the VS Code editor, choose the **Extensions** icon. This opens the Extensions view, which allows you to access the **VS Code Marketplace**.

  ![AWS Extension](images/aws-toolkit-extensions.png)

3. In the search box for **Extensions**, search for AWS Toolkit for Visual Studio Code. Choose the entry to see its details in the right pane.

4. In the right pane, choose **Install**.

5. Once installed, if you're prompted to restart the editor, choose **Reload Required** to finish installation.

6. Open the **Command Palette**, on the menu bar, choose **View**, **Command Palette**. Or use the following shortcut keys:
  - Windows and Linux – Press Ctrl+Shift+P.
  - macOS – Press Shift+Command+P.

7. Search for AWS and choose **AWS: Create Credentials Profile**.

8. Enter a name for the initial profile.

9. Enter the **Access key ID** from the credential file (.csv) you have downloaded earlier.

10. Enter the **Secret access key** from the credential file (.csv) you have downloaded earlier.

</details>

### 4. AWS Command Line Interface

Follow the [AWS CLI Getting Started](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv1.html) guide to install and configure the CLI on your machine. Use CLI version 1 for this workshop.

### 5. AWS SAM CLI
[AWS SAM CLI](https://docs.aws.amazon.com/lambda/latest/dg/test-sam-cli.html) is the AWS CLI tool for managing Serverless applications written with [Serverless Application Model (SAM)](https://github.com/awslabs/serverless-application-model).  SAM CLI can be used to test functions locally, start a local API Gateway from a SAM template, validate a SAM template, and generate sample payloads for various event sources. We will try using the SAM CLI to test functions and host our local API Gateway in this session.


#### Windows, Linux, macOS with pip [Recommended]
The easiest way to install **`sam`** is to use [pip](https://pypi.org/project/pip/). You must have [Python](https://www.python.org/) installed and added to your system's Environment path.

```bash
pip install aws-sam-cli
```

Verify the installation worked:

```bash
sam --version
```

#### Binary release

We also release the CLI as binaries that you can download and instantly use. You can find them under [Releases](https://github.com/awslabs/aws-sam-cli/releases) in the SAM CLI repo.


### 6. Docker
Running Serverless projects and functions locally with SAM CLI requires Docker to be installed and running. SAM CLI will use the `DOCKER_HOST` environment variable to contact the docker daemon.

* macOS: [Docker for Mac](https://store.docker.com/editions/community/docker-ce-desktop-mac)
* Windows: [Docker Toolbox](https://download.docker.com/win/stable/DockerToolbox.exe)
* Linux: Check your distro's package manager (e.g. yum install docker)

For macOS and Windows users: SAM CLI requires that the project directory (or any parent directory) is listed in Docker file sharing options.

Verify that docker is working, and that you can run docker commands from the CLI (e.g. `docker ps`). You do not need to install/fetch/pull any containers - SAM CLI will do it automatically as required.


## Modules

This workshop is broken up into multiple modules. You must complete each module before proceeding to the next.

0. [Lambda Applications](0_LambdaApp)
1. [Serverless Application Model (SAM)](1_ServerlessApplicationModel)
2. [AWS X-Ray Integration](2_XRay)
3. [Debugging Challenge](3_DebuggingChallenge)
