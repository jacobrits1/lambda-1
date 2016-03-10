# Environment

The base images strive to provide the same environment that AWS provides to
Lambda functions. This page describes it and any incompatibilities between AWS
Lambda and Dockerized Lambda.

## Request/Response

We originally built this to allow users to bring their Lambda functions to
IronWorker. IronWorker does not have request/response tasks, it only does async
communication. Hence none of the Request/Response workflows are currently
supported on any platform. `context.succeed()` will not do anything with result
on node.js, nor will returning anything from a Python function.

## Paths

We do not make any compatibility efforts towards running your lambda function
in the same working directory as it would run on AWS. If your function makes
such assumptions, please rewrite it.

## nodejs

* node.js version [0.10.42][nodev]. Thanks to Michael Hart for creating the
  smaller, Alpine Linux based image.
* ImageMagick version [6.9.3][magickv] and nodejs [wrapper 6.9.3][magickwrapperv]
* aws-sdk version [2.2.12][awsnodev]

[nodev]: https://github.com/mhart/alpine-node/blob/f025a0516b87e2a505c6be4ff2c7bf485a95dc5a/Dockerfile
[magickv]: https://pkgs.alpinelinux.org/package/main/x86_64/imagemagick
[magickwrapperv]: https://www.npmjs.com/package/imagemagick
[awsnodev]: https://aws.amazon.com/sdk-for-node-js/

### Event

Payloads MUST be a valid JSON object literal.

### Context object

* context.fail() does not currently truncate error logs.
* `context.functionName` is of the form of a docker image, for example
  `iron/test-function`.
* `context.functionVersion` is always the string `"$LATEST"`.
* `context.invokedFunctionArn` is not supported. Value is empty string.
* `context.memoryLimitInMB` does not reflect reality. Value is always `256`.
* `context.awsRequestId` reflects the environment variable `TASK_ID`. On local
  runs from `ironcli` this is a UUID. On IronWorker this is the task ID.
* `logGroupName` and `logStreamName` are empty strings.
* `identity` and `clientContext` are always `null`.

### Exceptions

If your handler throws an exception, we only log the error message. There is no
`v8::CallSite` compatible stack trace yet.

## Python 2.7

* CPython [2.7.11][pythonv]
* boto3 (Python AWS SDK) [1.2.3][botov].

[pythonv]: https://hub.docker.com/r/iron/python/tags/
[botov]: https://github.com/boto/boto3/releases/tag/1.2.3

### Event

Event is always a `__dict__` and the payload MUST be a valid JSON object
literal.

### Context object

* `context.functionName` is of the form of a docker image, for example
  `iron/test-function`.
* `context.functionVersion` is always the string `"$LATEST"`.
* `context.invokedFunctionArn` is `None`.
* `context.awsRequestId` reflects the environment variable `TASK_ID` which is
  set to the task ID on IronWorker. If TASK_ID is empty, a new UUID is used.
* `logGroupName`, `logStreamName`, `identity` and `clientContext` are `None`.

### Exceptions

If your Lambda function throws an Exception, it will not currently be logged as
a JSON object with trace information.

## Java 8

* OpenJDK Java Runtime [1.8.0][javav]

[javav]: https://hub.docker.com/r/iron/java/tags/

The Java8 runtime is significantly lacking at this piont and we **do not
recommend** using it.

### Context object

TODO