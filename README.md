![AWSM logo aws modules lambda api gateway JAWS](awsm_logo.png)

AWSM: Amazon Web Services Modules
=================================

**Amazon Web Services Modules (aws-modules)** are one or multiple AWS Lambda functions that perform specific tasks,
plus any AWS resource dependencies defined via AWS CloudFormation.  The purpose of aws-modules is to create an ecosystem of lambda functions
designed for re-use and easy installation into serverless applications.

aws-modules were designed to work with [JAWS: The Serverless AWS Framework](https://github.com/jaws-framework/JAWS).
The JAWS command line tool gives you handy commands to rapidly create and install aws-modules into your serverless applications.
View the JAWS documentation for more information.

## Structure

This is the directory structure of an aws-module with one lambda function:

```
awsm.json 			// Contains a "resources" property for the other resources required by this module and publishing information (name, author, etc.).
package.json		// For JS lambdas, include package.json here.  Require it in index.js
create				// Action/lambda level directory (Required)
	awsm.json 		// Contains a "lambda" property and a "endpoint" property for this lambda
	handler.js 		// Lambda function handler (JS example. Can be in any language AWS Lambda supports)
	index.js 	  	// Modular code you can require in this or other lambda functions (JS example).
```

This is the directory structure of an aws-module with multiple lambda functions:

```
awsm.json 			// Contains a "resources" property for the other resources required by this module and publishing information (name, author, etc.).
package.json 		// For JS lambdas, include package.json here.  Require it in index.js
create 				// Action/lambda level directory (Required)
	awsm.json 		// Contains a "lambda" property and a "endpoint" property for this lambda
	handler.js 		// Lambda function handler (JS example. Can be in any language AWS Lambda supports)
	index.js 	  	// Modular code you can require in this or other lambda functions.
show
	awsm.json
	handler.js
	index.js
update
	awsm.json
	handler.js
	index.js
delete
	awsm.json
	handler.js
	index.js
```
Remember, your lambda functions should be a thin wrapper around your own separate modules, to keep your code
testable, reusable, and AWS independent.  Basically, put as little code as you can in **handler.js** and as much code
as you can in **index.js** and additional files.

## JAWS CLI `module` command

The JAWS CLI [`module` command](https://github.com/jaws-framework/JAWS/blob/cf-deploy/docs/commands.md#jaws-module) is used to create,install, and upgrade awsm's.

## Configuration

aws-modules' configuration settings and dependencies are described in **awsm.json** files located in the module.

**Below are limited awsm.json examples.  To view all available properties see the [awsm.json file in this repo](./awsm.json).**

At the module's root, you should have an **awsm.json** file which describes publishing information as well as
AWS resources outside of the Lambda and API Gateway resources your module requires in the "resources" object:

```
"name": "usersCreate",
"description": "Creates a user",
"version": "0.0.1",
"location": "github.com/you/your_aws_modules_repo",
"resources": {
	"cloudFormation": {
		"LambdaIamPolicyDocumentStatements": [],
		"ApiGatewayIamPolicyDocumentStatements": [],
		"Resources": {}
	}
}
```
**Note** that JAWS defines some [CF parameters](https://github.com/jaws-framework/JAWS/blob/master/docs/project_structure.md#resources-cfjson) that can be used as `"Ref"`s

Within each resource/action directory is another **awsm.json** which describes either an AWS Lambda configuration,
an API Gateway configuration, or both.  awsm.json files within resource/action directories only need a "lambda" or
"apiGateway" property, or both.

```
"lambda": {
	"enVars": [],
	"package": {},
	"excludePatterns": {},
	"cloudFormation": {
		"Description": "",
		"MemorySize": 1024,
		"Runtime": "nodejs",
		"Timeout": 6
	}
},
"apiGateway": {
	"cloudFormation": {}
}
```


##### Lambda configuration options

**Note**: All of the attrs below assume the `lambda` attribute key prefix.

* `Handler,MemorySize,Runtime,Timeout`: can all be found in the [aws docs](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-function.html)
  * We recommend 1024 memory to improve cold/warm start times. `handler` is relative to back dir
* `envVars`: An array of environment variable names this project/module or lambda requires
* `deploy`: if true, this app will be deployed the next time the `jaws deploy --tags` is run. See `deploy` command docs for more info.
* `package`: How the code is packaged up into a zip file
  * `optimize`: How code is optimized for node runtimes, to improve lambda cold start time
    * `builder`: only `"browserify"` or `false` supported now.  If `false` will just zip up entire `back` dir
    * `minify`: js minify or not
    * `ignore`: array of node modules to ignore. See [ignoring](https://github.com/substack/browserify-handbook#ignoring-and-excluding)
    * `exclude`: array of node modules to exclude.  These modules will be loaded externally (from within zip or inside lambda).  Note `aws-sdk` for node [can not be browserified](https://github.com/aws/aws-sdk-js/issues/696). See [ignoring](https://github.com/substack/browserify-handbook#ignoring-and-excluding)
    * `includePaths`: Paths rel to back (dirs or files) to be included in zip. Paths included after optimization step.
  * `excludePatterns`: Array of regular expressions rel to back. Removed before optimization step. If not optimizing, everything in back dir will be included in zip. Use this to exclude stuff you don't want in your zip.  Strings will be passed to `new RegExp()`

For an optimize example using the most popular node modules see [browserify tests](../tests/test-prj/back/aws_modules/bundle/browserify)

For non optimize example see [non optimized tests](../tests/test-prj/back/aws_modules/bundle/nonoptimized)

##### API Gateway attributes:

TODO

## AWSM's

The aws-module format is only a few weeks old and we don't have a registry yet.  For the time being, here is a list of aws-modules:

####awsm-users - https://github.com/dekz/awsm-users

Creates REST API routes creating and authenticating users for your application via DynamoDB.

####awsm-cloudfront - https://github.com/boushley/awsm-cloudfront

This allows for hosting a web application entirely using JAWS.  Puts an AWS Cloudfront distribution in front of a static S3 bucket and API Gateway endpoints all on the same domain to avoid any same origin problems.  No need for CORS!

####image-resize - https://github.com/awsm-org/img-resize

Creates a REST API route for creating an image thumbnail that will resize any image URL submitted. This exercises many of the aws-module configurations. It also showcases how to browserify your lambda code and still include a non-optimized ver of aws-sdk.



