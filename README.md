![AWSM logo aws modules lambda api gateway JAWS](awsm_logo.png)

AWSM: The AWS-Module System
=================================

**aws-modules** contain one or multiple AWS Lambda functions,
plus their AWS resource dependencies defined via AWS CloudFormation.

The purpose of aws-modules is to encourage the development of lambda functions
designed for re-use and easy installation into applications.

## Structure

This is the directory structure of an aws-module with 1 lambda function:

```
awsm.json // Contains a "resources" property for the other resources required by this module
lambdas // The modules lambda function(s)
  users // Resource level directory (Required)
    create // Action level directory (Required)
      awsm.json // Contains a "lambda" property and a "endpoint" property for this lambda
      index.js // Lambda code. Can be in any language AWS Lambda supports
```

This is the directory structure of an aws-module with multiple lambda functions:

```
awsm.json // Contains a "resources" property for the other resources required by this module
lambdas // The modules lambda function(s)
  users // Resource level directory (Required)
    create // Action level directory (Required)
      awsm.json // Contains a "lambda" property and a "endpoint" property
      index.js // Lambda code. Can be in any language AWS Lambda supports
    show // Action level directory (Required)
      awsm.json // Contains a "lambda" property and a "endpoint" property
      index.js // Lambda code. Can be in any language AWS Lambda supports
    update // Action level directory (Required)
      awsm.json // Contains a "lambda" property and a "endpoint" property
      index.js Lambda code. Can be in any language AWS Lambda supports
    delete // Action level directory (Required)
      awsm.json // Contains a "lambda" property and a "endpoint" property
      index.js Lambda code. Can be in any language AWS Lambda supports
lib // Contains code shared across all lambda functions
 	users-dynamodb.js // Shared code
```
