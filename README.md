![AWSM logo aws modules lambda api gateway JAWS](awsm_logo.png)

AWSM: Amazon Web Services Modules
=================================

* **[Intro](https://github.com/awsm-org/awsm#intro)**
* **[awsm.json](https://github.com/awsm-org/awsm#awsmjson)**
	* **[awsm.json At Module Root](https://github.com/awsm-org/awsm#awsmjson-at-module-root)**
	* **[awsm.json At Lambda Root](https://github.com/awsm-org/awsm#awsmjson-at-lambda-root)**
	* **[Lambda Configuration Options](https://github.com/awsm-org/awsm#lambda-configuration-options)**
* **[Creating AWS-Modules](https://github.com/awsm-org/awsm#creating-aws-modules)**
* **[Creating Reusable AWS-Modules](https://github.com/awsm-org/awsm#creating-re-usable-aws-modules)**
	* **[AWSM + NPM-Modules](https://github.com/awsm-org/awsm#awsm--npm-modules)**
		* **[Architecture](https://github.com/awsm-org/awsm#architecture)**
		* **[Workflow](https://github.com/awsm-org/awsm#workflow)**
* **[Ideas & Themes](https://github.com/awsm-org/awsm#ideas--themes)**
* **[Registry](https://github.com/awsm-org/awsm#registry)**

## Intro

**Amazon Web Services Modules (aws-modules, awsm’s)** contain pre-written, isolated functions ready to run on one or multiple AWS Lambda functions. Some examples are: *functions that send out emails, register users or handle webhooks from other services*.

The purpose of the *aws-module* format is to create an ecosystem of re-usable, standardized, optimized Lambda functions ready for deployment and easy installation into serverless projects.

*aws-modules* were designed to work with [JAWS: The Serverless AWS Framework](https://github.com/jaws-framework/JAWS).
The JAWS command line tool comes with commands to create and install *aws-modules* into your serverless projects.  View the [JAWS documentation](https://github.com/jaws-framework/JAWS/tree/master/docs) for more information.

*aws-modules* will soon support all of the languages AWS Lambda supports.  Currently, only javascript (node.js) is supported.  Building a module system that supports multiple programming languages is challenging, but since the functions of serverless projects/applications are completely isolated, functions written in different programming languages can be combined within the same project.  Given some languages are more efficient for specific tasks, this is a nice benefit.

## awsm.json

#### awsm.json At Module Root

[Example of an `awsm.json` at the root of an aws-module](examples/awsm_root/awsm.json)

The defining feature of an *aws-module* is an `awsm.json` file located at the root of the module.  *aws-modules'* configuration settings, dependencies and authorship details are described in this `awsm.json` file.  Below, are limited `awsm.json` examples.  To view all available properties, see the [awsm.json file in this repo](./awsm.json).

Any required AWS resources (e.g., DynamoDB, S3) **outside** of the Lambda and API Gateway resources your module requires in the "resources" object:

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

Please note that JAWS defines some standard [ AWS CloudFormation parameters](https://github.com/jaws-framework/JAWS/blob/master/docs/project_structure.md#resources-cfjson) that can be used as `"Ref"`'s

#### awsm.json At Lambda Root

[Example of an `awsm.json` at the root of a lambda sub-folder](examples/awsm_lambda/awsm.json)

Within each resource/lambda directory is another **awsm.json** which describes either an AWS Lambda configuration,
an API Gateway configuration, or both.  `awsm.json` files within resource/action directories need only a "lambda" or
"apiGateway" property (or both).

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


#### Lambda Configuration Options

**Note**: All of the attrs below assume the `lambda` attribute key prefix.

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
* `cloudFormation`
  * `Handler,MemorySize,Runtime,Timeout`: can all be found in the [aws docs](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-function.html)
    * We recommend 1024 memory to improve cold/warm start times. `handler` is relative to back dir


## Creating AWS-Modules

Whether you are writing aws-modules exclusively for your serverless project (every Lambda function in a JAWS project is an aws-module) or writing them for sharing publicly, the process to create them is the same.

* Install the JAWS command line tool via npm, which you will use to generate scaffolding for new aws-modules:

		$ npm install jaws-framework -g

* Create a new JAWS project:

		$ jaws project create

* Create a new aws-module.  Enter a module name and the name of the first Lambda function your aws-module will use.  For example, your module name could be `awsm-images` and the first Lambda function your module offers is the ability to re-size images, enter the following:

		$ jaws module create awsm-images resize

* Scaffolding should now be available in your project’s `aws_modules` folder.

## Creating Re-Usable AWS-Modules

If you want to use your aws-module across multiple projects and optionally share it with the aws-module community, do the following:

* aws-modules rely on different package managers for delivery/installation.  Choose the package manager that corresponds with the runtime of the Lambda functions within your module and create a corresponding package.  Currently, only the nodejs runtime is supported by aws-modules.  Therefore, run `npm init` within the root of your aws-module to make your aws-module an npm-module and edit the `package.json` with your authorship information in your module’s root.

* Publish your aws-module to the respective package manager’s registry.  For npm-modules, run `npm publish`.

* Fill in your authorship details in your aws-module’s root awsm.json file.  Sorry for the redundancy, but we will be unveiling a separate awsm registry which will assist in search and discovery of aws-modules across all programming languages/AWS Lambda runtimes.

#### AWSM + NPM-Modules

##### Architecture

awsm npm modules should have the following structure.  Upon `npm install yourmodule`, JAWS will copy contents of the `awsm` folder into the serverless project's `aws_modules` folder on the npm *postinstall* hook.

```
module
	awsm.json
	package.json
	awsm
		lambda1
			awsm.json
			handler.js
			index.js
			event.json
	lib
		modularcode.js
```

###### `/awsm.json`
This contains module and authorship information for publishing your awsm on the upcoming aws-module registry.
###### `/package.json.json`
This contains module and authorship information for publishing your awsm as an npm module on their registry.  Remember, awsm's use different package managers for delivery.
###### `/awsm/`
This contains starter scaffolding for lambda functions and endpoints.  When someone installs your awsm via npm, the contents of this folder will be copied to their *aws_modules* folder, while the rest of your awsm will reside in their project's *node_modules* folder.  As a best practice, don't put a lot of code in the scaffolding.  Instead, focus on making your code modular and re-usable by putting the bulk of it in `lib/yourcode.js`.
###### `/awsm/lambda1`
Your awsm can have one or multiple lambda functions.  Each lambda function resides in a sub-folder like this.
###### `/awsm/lambda1/awsm.json`
This contains configuration settings for this lambda and/or a URL endpoint that will be created via API Gateway.  Lambda and API Gateway config information is stored in AWS CloudFormation syntax.
###### `/awsm/lambda1/hander.js`
This contains the handler which AWS Lambda will call.  As a best practice, don't put your logic in here.  Keep it separate so that it is re-usable, testable, and AWS independent.
###### `/awsm/lambda1/index.js`
This is your lambda's code, kept separate to be re-usable across modules, testable, and AWS independent.  Please note that all of the dependencies in your awsm's package.json can only be required in code that is in `/lib/`.  Do not require those dependencies in this `index.js` file or the `handler.js` file.
###### `/awsm/lambda1/event.json`
This is sample event information which you can use to test your Lambda function locally.
###### `/lib/`
When your awsm npm module is installed via JAWS, everything in the *awsm* folder will be copied to the project's *aws_modules* folder, but everything in this *lib* folder will be kept within the project's *node_modules* folder. Generally, code that should not be modified and code that needs to be required across all aws_modules in a serverless project should be put in this folder.  Code that will be modified should be put in the `awsm` folder.
###### `/lib/modularcode.js`
This is code that should not be modified (like a normal npm module) and can be shared/required across all aws_modules in a serverless project.  Please note that all dependencies that are in your awsm's package.json can only be required in code that is located in the `lib` folder.

##### Workflow

While you develop your module locally, it's useful to have a project to implement and test it in.  With the following steps, you can install the module in your project, after which it can be tested.

* Switch to the project in which you want to install your NPM module.

* Run `npm link <module-name>` to create a link in your project's `/node_modules` directory to the globally-installed link.

* Run `jaws postinstall <module-name> npm` to execute the `postinstall` command (since `npm link` skips the `postinstall` script).

Now your module is installed!

Please note that the module is not yet published to the NPM registry and therefore not yet added to your project's package.json.  Read more about this in the NPM documentation on [publishing NPM packages](https://docs.npmjs.com/getting-started/publishing-npm-packages).

## Ideas & Themes

So many things can be aws-modules.  Here are some most-demanded themes for inspiration:

* **Users/Sessions CRUD** - Bonus if you include sign on via popular 3rd party services (e.g., Github)
* **Analytics/Event Functions** - Captures common user events
* **Image functions** - Resize, filters, etc.
* **Email - Transactional** email functions (e.g., receipt, welcome, where you been?)
* **Email - Marketing** - Blasting html emails to many addresses for dirt cheap via Lambda + SES
* **Stripe Webhook Handlers**
* **Slack Webhook handlers**
* **Webhook handlers for all the things**
* **Twilio functions** - Send SMS
* **3rd Party Authentication Handlers** - Pre-built authentication with the most popular peeps.
* **Proxy Modules** - API Gateway has the ability to be a proxy for other endpoints and do request and response transforms without ever needing lambda.  This presents an opportunity for people to make their own API endpoints that mask other API endpoints and publish those as aws-modules.  For example, you can mask a Slack API endpoint with api.yourapp.com/slack/post_message and define how the response from Slack should be transformed in your awsm.apiGateway.cloudFormation template and publish that as a module for others to reuse.

## Registry

Want to tell the world about your aws module? Send a pull request against [registry.json](./registry.json). This poor mans registry is just a temporary solution.

