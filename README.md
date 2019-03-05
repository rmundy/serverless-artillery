# Serverless-artillery [![Build Status](https://travis-ci.org/Nordstrom/serverless-artillery.svg?branch=master)](https://travis-ci.org/Nordstrom/serverless-artillery) [![Coverage Status](https://coveralls.io/repos/github/Nordstrom/serverless-artillery/badge.svg?branch=master)](https://coveralls.io/github/Nordstrom/serverless-artillery?branch=master)

[//]: # (Thanks to https://www.divio.com/en/blog/documentation/)


# Introduction
Combine [`serverless`](https://serverless.com) with [`artillery`](https://artillery.io) and you get `serverless-artillery` (a.k.a. `slsart`). 

Serverless-artillery makes it easy to test your services for load and functionality quickly, with almost no code and without having to maintain any servers or testing infrastructure.

### Use serverless-artillery if
1. You want to know if your services (either internal or public) can handle different amount of traffic load (i.e. performance or load testing).
1. You want to test if your services behave as you expect after you deploy new changes (i.e. acceptance testing).
1. You want to constantly monitor your services over time to make sure the latency of your services is under control (i.e. monitoring mode).

# Table of Contents
<details><summary>Click to expand/collapse</summary>
<p>

- [Installation](#installation)
  - [Installing on local machine](#installing-on-local-machine)
    - [Prerequisite](#prerequisite)
      - [1. Node JS](#1-node-js)
      - [2. Serverless Framework CLI](#2-serverless-framework-cli)
    - [Installing serverless-artillery](#installing-serverless-artillery)
  - [Installing in Docker](installing-in-docker)
- [Uninstallation](#uninstallation)
- [How it works?](#how-it-works)
  - [Load generating Lambda function on AWS](#load-generating-lambda-function-on-aws)
- [Before running serverless-artillery](#before-running-serverless-artillery)
  - [Setup for Nordstrom Technology](#setup-for-nordstrom-technology)
  - [Setup for everyone else](#setup-for-everyone-else)
- [Tutorial 1: Run a quick performance test](#tutorial-1-run-a-quick-performance-test)
  - [T1.1. Setup AWS account credentials](#t11-setup-aws-account-credentials)
  - [T1.2. Deploy](#t12-deploy)
  - [T1.3. Invoke](#t13-invoke)
  - [T1.4. Remove](#t14-remove)
- [Tutorial 2: Performance test with custom script](#tutorial-2-performance-test-with-custom-script)
  - [T2.1. Create new directory](#t21-create-new-directory)
  - [T2.2. Create `script.yml`](#t22-create-scriptyml)
  - [T2.3. Understanding `script.yml`](#t23-understanding-scriptyml)
  - [T2.4. Customizing `script.yml`](#t24-customizing-script.yml)
  - [T2.5. Setup AWS account credentials](#t25-setup-aws-account-credentials)
  - [T2.6. Deploy assets to AWS](#t26-deploy-assets-to-aws)
  - [T2.7. Invoke performance test](#t27-invoke-performance-test)
  - [T2.8. Remove assets from AWS](#t28-remove-assets-from-aws)
- [Tutorial 3: Performance test with custom deployment assets](#tutorial-3-performance-test-with-custom-deployment-assets)
  - [T3.1. Create new directory](#t31-create-new-directory)
  - [T3.2. Create `script.yml`](#t32-create-scriptyml)
  - [T3.3. Understanding `script.yml`](#t33-understanding-scriptyml)
  - [T3.4. Customizing `script.yml`](#t34-customizing-script.yml)
  - [T3.5. Create custom deployment assets](#t35-create-custom-deployment-assets)
  - [T3.6. Understanding `serverless.yml`](#t36-understanding-serverlessyml)
    - [Service name](#service-name)
    - [Load generating Lambda function name](#load-generating-lambda-function-name)
    - [Load generating Lambda function permissions](#load-generating-lambda-function-permissions)
  - [T3.7. Customizing `serverless.yml`](#t37-customizing-serverlessyml)
    - [Service name](#service-name-1)
    - [Plugins](#plugins)
    - [Customization for Nordstrom Engineers](#customization-for-nordstrom-engineers)
  - [T3.8. Setup AWS account credentials](#t38-setup-aws-account-credentials)
  - [T3.9. Deploy assets to AWS](#t39-deploy-assets-to-aws)
  - [T3.10. Invoke performance test](#t310-invoke-performance-test)
  - [T3.11. Remove assets from AWS](#t311-remove-assets-from-aws)
- [Troubleshooting](#troubleshooting)
  - [Problems installing?](#problems-installing)
  - [Error: npm ERR! code EACCES](#error-npm-err-code-eacces)
- [Glossary](#glossary)
  - []()
  - []()
  - []()
  - []()
  - []()

</p>
</details>

# Installation

## Installing on local machine
You can install serverless-artillery on your local machine as follows.
### Prerequisite
#### 1. Node JS
Before installing serverless-artillery, install Node JS from https://nodejs.org/en/download/ or with your operating system’s package manager. You can install the latest LTS version. We support any version higher than maintenance LTS (v6+).
#### 2. Serverless Framework CLI
Before installing serverless-artillery, install Serverless Framework CLI (a.k.a. Serverless) (v1.38+). It should be either installed globally or available in the local node_modules. To install globally use the following command.
```
npm install -g serverless
```

### Installing serverless-artillery
Now you can install serverless-artillery on your local machine using the following command.
```
npm install -g serverless-artillery
```
To check that the installation succeeded, run:
```
slsart --version
```
You should see serverless-artillery print its version if the installation has been successful.

## Installing in Docker
If you prefer using Docker, refer to [example Dockerfile](Dockerfile) for installation. Please note that, post installation causes permission issues when installing in a Docker image. To successfully install in Docker make sure to add the following to your Dockerfile before the serverless and serverless-artillery install.
```
ENV NPM_CONFIG_PREFIX=/home/node/.npm-global
ENV PATH=$PATH:/home/node/.npm-global/bin
```

# Uninstallation
When needed, you can uninstall serverless-artillery from you local machine using the following command.
```
npm uninstall -g serverless-artillery
```

# How it works?
<img src="docs/HowItWorks.jpg" width="442">

* Serverless-artillery would be installed and run on your local machine.
* It would take your JSON or YAML load script (`script.yml`) that specifies 
  * test target/URL/endpoint/service, 
  * load progression,
  * and the scenarios that are important for your service to test.
* When you run `slsart deploy`, serverless-artillery would deploy a **load generating Lambda function called `serverless-artillery-<optional-unique-string-><stage default:dev>-loadGenerator`** on your AWS account along with other assets.
* Running the tests
  * **Performance test:** When you run `slsart invoke`, serverless-artillery would invoke the function.
    * It would generate the number of requests as specified in `script.yml` to specified test target in order to run the specified scenarios.
  * **Acceptance test:**
  * **Monitoring:**
* When you run `slsart remove`, serverless-artillery would remove these assets from your AWS account.

## Load generating Lambda function on AWS
<details><summary>Click to expand/collapse</summary>
<p>

### Artillery.io
- [Artillery.io](https://artillery.io/) (built by Hassy Veldstra of shoreditch-ops) is an existing open-source node package built for easy load testing and functional testing of an endpoint/URL.
- It takes in a developer-friendly JSON or YAML load script that specifies 
  - target/URL/endpoint, 
  - load progression,
  - and the scenarios that are important for your service to test.
- It generates specified load, and measures and reports the resulting latency and return codes.
- It generates the load by running on your local machine or servers. It is not a serverless load generation tool.

### Load generating Lambda

<img src="docs/Architecture.gif">

- Serverless-artillery generates the requests to run the specified tests using load generating Lambda function named `serverless-artillery-<optional-unique-string-><stage default:dev>-loadGenerator` that is deployed and invoked on AWS along with other assets.
- It has an ephimeral architecture. It only exists as long as you need it.
- It runs Artillery.io node package in AWS Lambda function.
  - Each lambda function can only generate a certain amount of load, and can only run for up to five minutes (five minutes was a built-in limitation of AWS Lambda) (now 15 minutes). 
  - Given these limitations, it is often necessary to invoke more lambdas - both to scale horizontally (to generate higher load) as well as handing off the work to a new generation of lambdas before their run-time has expired.
- Above diagram shows how Serverless Artillery solves this problem.
  - It first runs the Lamdba function in a **control** mode. It examines the submitted load config JSON/YAML script (this is identical to the original “servered” [Artillery.io](https://artillery.io/) script). If the load exceeds what a single lambda is configured to handle, then the load config is chopped up into workloads achievable by a single lambda. 
  - Control lambda then invokes as many **worker** lambdas as necessary to generate the load. 
  - Towards the end of the Lambda runtime the controller lambda invokes a new controller lambda to produce load for the remaining duration.
- The result of the load test can be reported to CloudWatch, InfluxDB or Datadog through plugins and then visualized with CloudWatch, Grafana or Datadog dashboard.

</p>
</details>

# Before running serverless-artillery
Serverless-artillery needs to _deploy_ assets like [load generating Lambda function](docs/LoadGeneratorLambda.md) to AWS, _invoke_ the function to run the tests and _remove_ these assets from AWS when not needed. Hence you need an AWS account and setup credentials with which to deploy, invoke and remove the assets from AWS.

## Setup for Nordstrom Technology
If you are a **_Nordstrom_** engineer, please see the page titled **_`Serverless Artillery - Nordstrom Technology Setup`_** in **Confluence** and follow the instructions there.
## Setup for everyone else
In order to use serverless-artillery, depending on the AWS account environment you're working in, you may need to define `AWS_PROFILE` to declare the AWS credentials to use and possibly `HTTP_PROXY` in order to escape your corporate proxy.  See the [Serverless Framework docs](https://serverless.com/framework/docs/) or [serverless-artillery workshop](https://github.com/Nordstrom/serverless-artillery-workshop)'s [Lesson 0](https://github.com/Nordstrom/serverless-artillery-workshop/tree/master/Lesson0%20-%20Before%20the%20workshop) for details of how to set your system up for successful deployment, invocation, and removal. 

# Tutorial 1: Run a quick performance test
If you want to quickly test your setup or see serverless-artillery in action, do the following to quickly run a **small load/performance test**. Don't worry about what these commands do in detail. This document explains them in detail later.

### T1.1. Setup AWS account credentials
Make sure you have [setup your AWS account credentials](#before-running-serverless-artillery) before proceeding. **It should be running while using any serverless-artillery command that interacts with AWS.**

### T1.2. Deploy
The following command will deploy required assets (like [load generating Lambda function](docs/LoadGeneratorLambda.md)) to the AWS account you selected in the previous step.
```
slsart deploy
```
By default it uses AWS CloudFormation Stack name `serverless-artillery-dev`. You will see the stack created if you go to your AWS account console > CloudFormation.

### T1.3. Invoke
The following command will invoke [load generating Lambda function](docs/LoadGeneratorLambda.md) using default load script (`script.yml`), creating small traffic against the sample endpoint specified in the default script.
```
slsart invoke
```
At the end of the test serverless-artillery will generate a report of the test. **Please note that this report is generated only for small load.**

### T1.4. Remove
The following command will remove the AWS CloudFormation Stack deployed in step 1. If you are a **_Nordstrom_** engineer, please see the page titled **_`Serverless Artillery - Remove Instructions`_** in **Confluence** and follow the instructions there.
```
slsart remove
```

# Tutorial 2: Performance test with custom script
Throughout this tutorial we will walk you towards performance testing the AWS website, https://aws.amazon.com/.

We would test with our custom script but would use default deployment assets.

### T2.1. Create new directory
Start by creating a new directory for this tutorial and go to that directory in command line.

### T2.2. Create `script.yml`
Serverless-artillery needs to know information about the performance test that user wants to run. It needs information like, the target URL of the service that user wants to test, load progression, user's interaction with the service (scenarios) etc. All these are described in a `script.yml` file. It is the same `script.yml` that Artillery.io uses. 
- **Please see [here for basic concepts for Artillery.io usage](https://artillery.io/docs/basic-concepts/#basic-concepts).**
- **Please see [here for Artillery.io's test script reference](https://artillery.io/docs/script-reference/).**

Run the following command to create the initial `script.yml` file.
```
slsart script
```

### T2.3. Understanding `script.yml`
Open `script.yml` with your favorite editor to see what it contains.
<details><summary>Click to expand/collapse</summary>
<p>

```
# Thank you for trying serverless-artillery!
# This default script is intended to get you started quickly.
# There is a lot more that Artillery can do.
# You can find great documentation of the possibilities at:
# https://artillery.io/docs/
config:
  # this hostname will be used as a prefix for each URI in the flow unless a complete URI is specified
  target: "http://aws.amazon.com"
  phases:
    -
      duration: 5
      arrivalRate: 2
scenarios:
  -
    flow:
      -
        get:
          url: "/"

```

</p>
</details>

- The script has [`config` section](https://artillery.io/docs/script-reference/#the-config-section)
  - under which it specifies http://aws.amazon.com as the `target` for the test
    - and that requests should be made using [HTTP protocol](https://artillery.io/docs/http-reference/)
  - There is one [load `phase`](https://artillery.io/docs/script-reference/#load-phases) of `duration` of 5 sec and `arrivalRate` of 2 new virtual users arriving every second.
- The script has [`scenarios` section](https://artillery.io/docs/script-reference/#scenarios)
  - which contains one scenario
    - which contains one flow
      - which has one [flow action](https://artillery.io/docs/http-reference/#flow-actions) to send [GET request](https://artillery.io/docs/http-reference/#get-post-put-patch-delete-requests) for the specified `target`.

### T2.4. Customizing `script.yml`
This step is optional in the tutorial. If you like you can customize `script.yml` as follows.
- If you have a public endpoint/service/URL that you would like to load test then you can change `target` to point to that.
- You can also change the [load `phase`](https://artillery.io/docs/script-reference/#load-phases) and [`scenarios` section](https://artillery.io/docs/script-reference/#scenarios) as per your need. We recommend using a low load to try the tool first.

### T2.5. Setup AWS account credentials
Make sure you have [setup your AWS account credentials](#before-running-serverless-artillery) before proceeding. **It should be running while using any serverless-artillery command that interacts with AWS.**

### T2.6. Deploy assets to AWS
We need to deploy assets (like [load generating Lambda function](docs/LoadGeneratorLambda.md)) to your AWS account before we can use it to start our test.

Use the following command to deploy the assets.
```
slsart deploy
```
You can go to your AWS account console > CloudFormation, and see AWS stack `serverless-artillery-dev` created there if the command is successful.

### T2.7. Invoke performance test
Now you are all set to invoke performance test using following command.
```
slsart invoke
```
At the end of the test serverless-artillery will generate a report of the test. **Please note that this report is generated only for small load.**

**NOTE** that for performance testing, the command will take the `script.yml` from your local machine (and not the one deployed in AWS account) to run the performance test. Hence if you edit it on your local machine after deploying assets to AWS, you don't need to deploy again in order to run the performance test again. Also note that this is true only for performance test and acceptance test and not monitoring.

### T2.8. Remove assets from AWS
After the test is done, you can remove the assets from AWS using following command. If you are a **_Nordstrom_** engineer, please see the page titled **_`Serverless Artillery - Remove Instructions`_** in **Confluence** and follow the instructions there.
```
slsart remove
```

# Tutorial 3: Performance test with custom deployment assets

Throughout this tutorial we will walk you towards performance testing the AWS website, https://aws.amazon.com/.

We would test with our custom script and custom deployment assets.

### T3.1. Create new directory
Start by creating a new directory for this tutorial and go to that directory in command line.

### T3.2. Create `script.yml`
This section is same as before. See [here](#t22-create-scriptyml) for details.

Run the following command to create the initial `script.yml` file.
```
slsart script
```

### T3.3. Understanding `script.yml`
This section is same as before. See [here](#t23-understanding-scriptyml) for details.

### T3.4. Customizing `script.yml`
This section is same as before. See [here](#t24-customizing-script.yml) for details.

### T3.5. Create custom deployment assets
Create a local copy of the deployment assets for customization and deployment using following command.
```
slsart configure
```
The important files among other files created by this command are as follows.

|file|description|
|:----|:----------|
|`package.json`|Node.js dependencies for the load generator Lambda. Add Artillery.io plugins you want to use here.|
|`serverless.yml`|Serverless service definition. Change AWS-specific settings here.|
|`handler.js`|Load generator Lambda code. **EDIT AT YOUR OWN RISK.**|

### T3.6. Understanding `serverless.yml`
Open `serverless.yml` with your favorite editor to see what it contains.
<details><summary>Click to expand/collapse</summary>
<p>

```
# We're excited that this project has provided you enough value that you are looking at its code!
#
# This is a standard [Serverless Framework](https://www.serverless.com) project and you should
# feel welcome to customize it to your needs and delight.
#
# If you do something super cool and would like to share the capability, please open a PR against
# https://www.github.com/Nordstrom/serverless-artillery
#
# Thanks!

# If the following value is changed, your service may be duplicated (this value is used to build the CloudFormation
# Template script's name)
service: serverless-artillery-XnBa473psJ

provider:
  name: aws
  runtime: nodejs8.10
  iamRoleStatements:
    # This policy allows the function to invoke itself which is important if the script is larger than a single
    # function can produce
    - Effect: 'Allow'
      Action:
        - 'lambda:InvokeFunction'
      Resource:
        'Fn::Join':
          - ':'
          -
            - 'arn:aws:lambda'
            - Ref: 'AWS::Region'
            - Ref: 'AWS::AccountId'
            - 'function'
            - '${self:service}-${opt:stage, self:provider.stage}-loadGenerator*' # must match function name
    # This policy allows the function to publish notifications to the SNS topic defined below with logical ID monitoringAlerts
    - Effect: 'Allow'
      Action:
        - 'sns:Publish'
      Resource:
        Ref: monitoringAlerts # must match the SNS topic's logical ID
functions:
  loadGenerator: # !!Do not edit this name!!
    handler: handler.handler    # the serverlessArtilleryLoadTester handler() method can be found in the handler.js source file
    timeout: 300                # set timeout to be 5 minutes (max for Lambda)
    environment:
      TOPIC_ARN:
        Ref: monitoringAlerts
      TOPIC_NAME:
        'Fn::GetAtt':
          - monitoringAlerts
          - TopicName
    events:
      - schedule:
          name: '${self:service}-${opt:stage, self:provider.stage}-monitoring' # !!Do not edit this name!!
          description: The scheduled event for running the function in monitoring mode
          rate: rate(1 minute)
          ########################################################################################################################
          ### !! BEFORE ENABLING... !!!
          ### 0. Change `'>>': script.yml` below to reference the script you want to use for monitoring if that is not its name.
          ###    The script must be in this directory or a subdirectory.
          ### 1. Modify your `script.yml` to provide the details of invoking every important surface of your service, as per
          ###    https://artillery.io/docs
          ### 2. Add a `match` clause to your requests, specifying your expectations of a successful request.  This relatively
          ###    undocumented feature is implemented at: https://github.com/shoreditch-ops/artillery/blob/82bdcdfc32ce4407bb197deff2cee13b4ecbab3b/core/lib/engine_util.js#L318
          ###    We would welcome the contribution of a plugin replacing this as discussed in https://github.com/Nordstrom/serverless-artillery/issues/116
          ### 3. Modify the `monitoringAlerts` SNS Topic below, uncommenting `Subscription` and providing subscriptions for any
          ###    alerts that might be raised by the monitoring function.  (To help you out, we've provided commented-out examples)
          ###    (After all, what good is monitoring if noone is listening?)
          ### 4. Deploy your new assets/updated service using `slsart deploy`
          ### 5. [As appropriate] approve the subscription verifications for the SNS topic that will be sent following its creation
          ########################################################################################################################
          enabled: false
          input:
            '>>': script.yml
            mode: monitoring
resources:
  Resources:
    monitoringAlerts: # !!Do not edit this name!!
      Type: 'AWS::SNS::Topic'
      Properties:
        DisplayName: '${self:service} Monitoring Alerts'
#        Subscription: # docs at https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sns-subscription.html
#          - Endpoint: http://<host>/<path> # the endpoint is an URL beginning with "http://"
#            Protocol: http
#          - Endpoint: https://<host>/<path> # the endpoint is a URL beginning with "https://"
#            Protocol: https
#          - Endpoint: <target>@<host> # the endpoint is an email address
#            Protocol: email
#          - Endpoint: <target>@<host> # the endpoint is an email address
#            Protocol: email-json
#          - Endpoint: <phone-number> # the endpoint is a phone number of an SMS-enabled device
#            Protocol: sms
#          - Endpoint: <sqs-queue-arn> # the endpoint is the ARN of an Amazon SQS queue
#            Protocol: sqs
#          - Endpoint: <endpoint-arn> # the endpoint is the EndpointArn of a mobile app and device.
#            Protocol: application
#          - Endpoint: <lambda-arn> # the endpoint is the ARN of an AWS Lambda function.
#            Protocol: lambda
```

</p>
</details>

Please refer to [`serverless.yml` documentation](https://serverless.com/framework/docs/providers/aws/guide/serverless.yml/) for details. It contains assets needed for monitoring (turned off by default) as well which we will discuss later.
#### Service name
- In above `serverless.yml` the `service` name is set to `serverless-artillery-XnBa473psJ`. In your `serverless.yml` the string at the end (`XnBa473psJ`) would be different.
- This will be the AWS CloudFormation stack name when you run `slsart deploy`.
- The `slsart configure` command adds a random string at the end so you get a unique stack name that does not conflict with anyone else also deploying to the same AWS account.
- You can change `service` name to some other unique string as per your need like `serverless-artillery-<unique-string>`.
- The rest of the `serverless.yml` refers to the service name by using `${self:service}`.
#### [Load generating Lambda function](#load-generating-lambda-function-on-aws) name
The Serverless framework automatically names the Lambda function based on the service, stage and function name as follows.
- The function `loadGenerator` when deployed is named as `${self:service}-${opt:stage, self:provider.stage}-loadGenerator`.
  - `${self:service}` is name of the service. In this `serverless.yml` it is `serverless-artillery-XnBa473psJ`.
  - `${opt:stage, self:provider.stage}` will either use `${opt:stage}` or `${self:provider.stage}`.
    - `${opt:stage}` refers to the (optional) stage name passed in `slsart deploy [--stage <stage-name>]` command.
    - If no stage name is passed in the deploy command then `${self:provider.stage}` would be used. It is the `stage` name set under `provider` section in the `serverless.yml`. If one is not provided (like in above example) it is set to `dev`. See [here](https://serverless.com/framework/docs/providers/aws/guide/serverless.yml/).
- In this example function name will be set to `serverless-artillery-XnBa473psJ-dev-loadGenerator` while running `slsart deploy` command (note no stage name specified).
#### [Load generating Lambda function](#load-generating-lambda-function-on-aws) permissions
- In order to generate load the load generating Lambda needs to invoke itself.
- The `iamRoleStatements` section in the `serverless.yml` gives the load generating Lambda function to invoke itself (`lambda:InvokeFunction`).

### T3.7. Customizing `serverless.yml`
This step is optional in the tutorial. If you like you can customize `serverless.yml` as follows.

#### Service name
- You can change `service` name to some other unique string as per your need like `serverless-artillery-<unique-string>`. This will be the AWS CloudFormation stack name when you run `slsart deploy`.

#### Plugins
You can customize the `serverless.yml` to use required tools/plugins mentioned [below](#addlink)**ASHMITODO**. 

In this tutorial you can add [artillery-plugin-cloudwatch](https://github.com/Nordstrom/artillery-plugin-cloudwatch) to record test results to AWS CloudWatch.
1. To allow the Lambda code to write to CloudWatch, the correct NPM package dependency must be added. This modifies the package.json file to include the necessary dependency.
```
npm install --save artillery-plugin-cloudwatch
```
2. Update the config portion of `script.yml` to add CloudWatch plugin from the example below:
```
config:
  plugins:
    cloudwatch:
      namespace: "<cloud-watch-namespace-example-serverless-artillery-XnBa473psJ-loadtest>"
```
3. In `serverless.yml`, under the following section (already exists)
```
provider:
  iamRoleStatements:
```
add the following
```
    - Effect: 'Allow'
      Action:
        - 'cloudwatch:PutMetricData'
      Resource:
        - '*'
```

#### Customization for Nordstrom Engineers
If you are a **_Nordstrom_** engineer, please see the page titled **_`Serverless Artillery - Nordstrom Technology Policies`_** in **Confluence** and follow the instructions there.

### T3.8. Setup AWS account credentials
Make sure you have [setup your AWS account credentials](#before-running-serverless-artillery) before proceeding. **It should be running while using any serverless-artillery command that interacts with AWS.**

### T3.9. Deploy assets to AWS
This section is same as before. See [here](#t26-deploy-assets-to-aws) for details.

### T3.10. Invoke performance test
This section is same as before. See [here](#t27-invoke-performance-test) for details.

If you used CloudWatch plugin you will be able to view the metrics on the CloudWatch dashboard. You can learn more about using CloudWatch dashboard [here](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Dashboards.html). **Note that it can take few minutes for the data to propogate to CloudWatch.**

### T3.11. Remove assets from AWS
This section is same as before. See [here](#t28-remove-assets-from-aws) for details.

# Troubleshooting
### Problems installing?
**ASHMITODO:Look into this:**
#### Error: npm ERR! code EACCES
If you are installing into a node_modules owned by root and getting error `npm ERR! code EACCES`, [read this](root-owns-node-modules.md).

# Glossary
- service/ end point/ target URL
- load/ traffic/ requests
- deploy to AWS
- AWS stack
- SA script
- load testing
- performance testing
- acceptance testing
- monitoring
- deployment assets. default vs custom
- local deployment assets and remote deployment assets
- 
