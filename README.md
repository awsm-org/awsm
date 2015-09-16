![AWSM logo aws modules lambda api gateway JAWS](awsm_logo.png)

AWSM: Amazon Web Services Modules
=================================

**Amazon Web Services Modules (aws-modules)** contain one or multiple AWS Lambda functions,
plus their AWS resource dependencies defined via AWS CloudFormation.

The purpose of aws-modules is to encourage the development of lambda functions
designed for re-use and easy installation into applications.

The specifications for aws-modules are below.

## Structure

This is the directory structure of an aws-module with 1 lambda function:

```
awsm.json 		// Contains a "resources" property for the other resources required by this module
lambdas 		// The modules lambda function(s)
  users 		// Resource level directory (Required)
    create 		// Action level directory (Required)
      awsm.json 	// Contains a "lambda" property and a "endpoint" property for this lambda
      index.js 		// Lambda code. Can be in any language AWS Lambda supports
```

This is the directory structure of an aws-module with multiple lambda functions:

```
awsm.json 			// Contains a "resources" property for the other resources required by module
lambdas 			// The modules lambda function(s)
  users 			// Resource level directory (Required)
    create 			// Action level directory (Required)
      awsm.json 		// Contains a "lambda" property and a "endpoint" property
      index.js 			// Lambda code. Can be in any language AWS Lambda supports
    show 			// Action level directory (Required)
      awsm.json 		// Contains a "lambda" property and a "endpoint" property
      index.js 			// Lambda code. Can be in any language AWS Lambda supports
    update 			// Action level directory (Required)
      awsm.json 		// Contains a "lambda" property and a "endpoint" property
      index.js			// Lambda code. Can be in any language AWS Lambda supports
    delete 			// Action level directory (Required)
      awsm.json 		// Contains a "lambda" property and a "endpoint" property
      index.js                  // Lambda code. Can be in any language AWS Lambda supports
lib 				// Contains code shared across all lambda functions
  users-dynamodb.js		// Shared code
```

## Configuration

aws-modules' configuration settings and dependencies are described in **awsm.json** files located in the module.

At the module's root, you should have an **awsm.json** file which describes publishing information as well as
AWS resources outside of the Lambda and API Gateway resources your module requires in the "resources" object:

```
"name": "usersCreate",
"description": "Creates a user",
"version": "0.0.1",
"location": "github.com/you/your_aws_modules_repo",
"resources": {
	"cloudFormation": {
		"PolicyDocuments": [ ],
		"Resources": { }
	}
}
```

Within each resource/action directory is another **awsm.json** which describes either an AWS Lambda configuration,
an API Gateway configuration, or both.  awsm.json files within resource/action directories only need a "lambda" or
"apiGateway" property, or both.

```
"lambda": {
	"cloudFormation": {
		"Description": "",
    "Handler": "",
    "MemorySize": 1024,
    "Runtime": "nodejs",
    "Timeout": 6
	}
},
"apiGateway": {
	"cloudFormation": {
		"Type": "AWS",
    "Path": "users/create",
    "Method": "POST",
    "AuthorizationType": "none",
    "ApiKeyRequired": false,
    "RequestTemplates": {},
    "RequestParameters": {},
    "Responses": {
    	"default": {
      	"statusCode": "200",
      	"responseParameters": {},
      	"responseTemplates": {
      	"application/json": ""
      }
    },
    "400": {
    	"statusCode": "400"
    }
  }
}
```