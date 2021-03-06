# Failure injection for AWS Lambda - failure-lambda

## Description

`failure-lambda` is a small Node module for injecting failure into AWS Lambda (https://aws.amazon.com/lambda). It offers a simple failure injection wrapper for your Lambda handler where you then can choose to inject failure by setting the `failureMode` to `latency`, `exception`, `denylist`, `diskspace` or `statuscode`. You control your failure injection using SSM Parameter Store.

## How to install

1. Install `failure-lambda` module using NPM.
```bash
npm install failure-lambda
```
2. Add the module to your Lambda function code.
```js
const failureLambda = require('failure-lambda')
```
3. Wrap your handler.
```js
exports.handler = failureLambda(async (event, context) => {
  ...
})
```
4. Create a parameter in SSM Parameter Store.
```json
{"isEnabled": false, "failureMode": "latency", "rate": 1, "minLatency": 100, "maxLatency": 400, "exceptionMsg": "Exception message!", "statusCode": 404, "diskSpace": 100, "denylist": ["s3.*.amazonaws.com", "dynamodb.*.amazonaws.com"]}
```
```bash
aws ssm put-parameter --region eu-west-1 --name failureLambdaConfig --type String --overwrite --value "{\"isEnabled\": false, \"failureMode\": \"latency\", \"rate\": 1, \"minLatency\": 100, \"maxLatency\": 400, \"exceptionMsg\": \"Exception message!\", \"statusCode\": 404, \"diskSpace\": 100, \"denylist\": [\"s3.*.amazonaws.com\", \"dynamodb.*.amazonaws.com\"]}"
```
5. Add an environment variable to your Lambda function with the key FAILURE_INJECTION_PARAM and the value set to the name of your parameter in SSM Parameter Store.
6. Try it out!

## Usage

Edit the values of your parameter in SSM Parameter Store to use the failure injection module.

* `isEnabled: true` means that failure is injected into your Lambda function.
* `isEnabled: false` means that the failure injection module is disabled and no failure is injected.
* `failureMode` selects which failure you want to inject. The options are `latency`, `exception`, `denylist`, `diskspace` or `statuscode` as explained below.
* `rate` controls the rate of failure. 1 means that failure is injected on all invocations and 0.5 that failure is injected on about half of all invocations.
* `minLatency` and `maxLatency` is the span of latency in milliseconds injected into your function when `failureMode` is set to `latency`.
* `exceptionMsg` is the message thrown with the exception created when `failureMode` is set to `exception`.
* `statusCode` is the status code returned by your function when `failureMode` is set to `statuscode`.
* `diskSpace` is size in MB of the file created in tmp when `failureMode` is set to `diskspace`.
* `denylist` is an array of regular expressions, if a connection is made to a host matching one of the regular expressions it will be blocked.

## Example

In the subfolder `example` is a simple Serverless Framework template which will install a Lambda function and a parameter in SSM Parameter Store.
```bash
npm i failure-lambda
sls deploy
```

## Notes

Inspired by Yan Cui's articles on latency injection for AWS Lambda (https://hackernoon.com/chaos-engineering-and-aws-lambda-latency-injection-ddeb4ff8d983) and Adrian Hornsby's chaos injection library for Python (https://github.com/adhorn/aws-lambda-chaos-injection/).

## Changelog

### 2020-08-24 v0.3.0

* Changed mitm mode from connect to connection for quicker enable/disable of failure injection.
* Renamed block list failure injection to denylist (breaking change for that failure mode).
* Updated dependencies.

### 2020-02-17 v0.2.0

* Added block list failure.
* Updated example application to store file in S3 and item in DynamoDB.

### 2020-02-13 v0.1.1

* Fixed issue with exception injection not throwing the exception.

### 2019-12-30 v0.1.0

* Added disk space failure.
* Updated example application to store example file in tmp.

### 2019-12-23 v0.0.1

* Initial release

## Contributors

**Gunnar Grosch** - [GitHub](https://github.com/gunnargrosch) | [Twitter](https://twitter.com/gunnargrosch) | [LinkedIn](https://www.linkedin.com/in/gunnargrosch/)

**Jason Barto** - [GitHub](https://github.com/jpbarto) | [Twitter](https://twitter.com/Jason_Barto) | [LinkedIn](https://www.linkedin.com/in/jasonbarto)
