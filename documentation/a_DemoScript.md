# Deloitte Engineering Day - AWS CDK Demo

## Agenda

-------

## Examples 

### `hello-world` 
This will be a ? step process by which you will create the following: 

{{ Insert architecture pattern}}

--------

#### 1. Init an AWS CDK Project
1. From root dir, create a dir called `hello-world` and nav to the `hello-world` dir

2. In the `hello-world` dir, run the following command 
```s
cdk init --language typescript
```

<p align="center">
<img width="450" alt="image" src="https://github.com/user-attachments/assets/eae6f956-8f9a-48e9-9f9d-e22497a3796b">
</p>

This command results in a handful of activities that AWS CDK will execute for you: 
1. It will initiate a dir structure for your AWS CDK project 
- [ ] the `bin` folder is home for your `hello-world` application
- [ ] the `lib` folder will reference the *App*, and is where you will store your *Stack(s)*
- [ ] the `test` folder will house test cases for your AWS resources
- [ ] ancillary files associated with a Typescript / NodeJS project (e.g. package.json, package-lock.json, tsconfig.json, .gitignore)

<p align="center">
<img width="326" alt="image" src="https://github.com/user-attachments/assets/a2b972f0-f0f7-4ff5-9139-847d60852726">
</p>

> NOTE: The directory must be empty or you will get an error
> OPTIONS: The AWS CDK provides mulitple language support. Options include 1. _Typescript_ 2. _Javascript_ 3. _Python_ 4. _Java_ 5. _C#_ & 6. _Go_ currently. 

--------

#### 2. Create a Lambda function

1. Create a sub-dir within the `hello-world` example called `lambda` and create a file called `hello.js`

```s
mkdir lambda && cd lambda
touch hello.js
```

<p align="center">
<img width="347" alt="image" src="https://github.com/user-attachments/assets/97a0c25a-2161-4f52-8c0f-fa4afe84dae0">
</p>

2. Input the following code in your `hello.js` file 

```js
exports.handler = async (event) => {
    return {
        statusCode: 200,
        headers: { "Content-Type": "text/plain" },
        body: JSON.stringify({ message: "Deloitte Engineering Day ... I want to welcome you to the world of AWS CDK!" }),
    };
};
```

--------

#### 3. Define your Lambda Construct 

1. Open the project file that defines your CDK stack -- this is the `lib/hello-world-stack.ts` file. You will modify this file to define your constructs. The following is an example of your starting stack file:

```js
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
// import * as sqs from 'aws-cdk-lib/aws-sqs';

export class HelloWorldStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // The code that defines your stack goes here

    // example resource
    // const queue = new sqs.Queue(this, 'HelloWorldQueue', {
    //   visibilityTimeout: cdk.Duration.seconds(300)
    // });
  }
}
```

> NOTE: In this file, the AWS CDK is doing the following: 1. Your CDK stack instance is instantiated from the `Stack` class. 2. The Constructs base class is imported and provided as the scope or parent of your stack instance.

2. Update this code to the following: 

```js
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
// Import Lambda L2 construct
import * as lambda from 'aws-cdk-lib/aws-lambda';

export class CdkHelloWorldStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Define the Lambda function resource
    const helloWorldFunction = new lambda.Function(this, 'HelloWorldFunction', {
      runtime: lambda.Runtime.NODEJS_20_X, // Choose any supported Node.js runtime
      code: lambda.Code.fromAsset('lambda'), // Points to the lambda directory
      handler: 'hello.handler', // Points to the 'hello' file in the lambda directory
    });
  }
}
```

> NOTE: What this code is doing is importing and using the `aws-lambda` *L2 construct* from the AWS Construct Library. 
> `runtime` – The environment the function runs in. Here, we use Node.js version 20.x.
> `code` – The path to the function code on your local machine.
> `handler` – The name of the specific file that contains your function code.

---------

#### 4. Define your API Gateway Construct

1. Return to your `lib/hello-world-stack.ts` file. You will modify this file to define an additional construct - an 1. API Gateway resource & 2. your API Gateway route to our `helloWorldFunction`. Add the following to the file: 

First, at the top of your file add an `import` statement to bring in the AWS CDK Construct library for the API Gateway. 
```js
// ...
import * as apigateway from 'aws-cdk-lib/aws-apigateway';
```

Then, below the existing code, inpu the following API Gateway construct and API Gateway route configuraiton. 
```js
// ... 
    // Define the API Gateway resource
    const api = new apigateway.LambdaRestApi(this, 'HelloWorldApi', {
      handler: helloWorldFunction,
      proxy: false,
    });
        
    // Define the '/hello' resource with a GET method
    const helloResource = api.root.addResource('hello');
    helloResource.addMethod('GET');
// ...
```

Your overall `hello-world-stack.ts` file should now look like this: 

```js
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
// Import Lambda L2 construct
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as apigateway from 'aws-cdk-lib/aws-apigateway';

export class CdkHelloWorldStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Define the Lambda function resource
    const helloWorldFunction = new lambda.Function(this, 'DeloitteEngineering_HelloWorldFunction', {
      runtime: lambda.Runtime.NODEJS_20_X, // Choose any supported Node.js runtime
      code: lambda.Code.fromAsset('lambda'), // Points to the lambda directory
      handler: 'hello.handler', // Points to the 'hello' file in the lambda directory
    });

    // Define the API Gateway resource
    const api = new apigateway.LambdaRestApi(this, 'HelloWorldApi', {
      handler: helloWorldFunction,
      proxy: false,
    });
        
    // Define the '/hello' resource with a GET method
    const helloResource = api.root.addResource('hello');
    helloResource.addMethod('GET');Construct
  }
}
```

> NOTE: Here, you create an API Gateway REST API resource, along with the following:
> An integration between the REST API and your Lambda function, allowing the API to invoke your function. This includes the creation of a Lambda permission resource.
> A new resource or path named hello that is added to the root of the API endpoint. This creates a new endpoint that adds /hello to your base URL.
> A GET method for the hello resource. When a GET request is sent to the /hello endpoint, the Lambda function is invoked and its response is returned.

---------

#### 5. Prepare your application for deployment

1. Run the following command to build the `npm project`

```s
npm run build
```
<p align="center">
<img width="450" alt="image" src="https://github.com/user-attachments/assets/36e5c5a8-1cc7-423e-ab49-572d89008d19">
</p>

2. Verify that you know where you are going to deploy your resources too. Take a peek at your config file 

<p align="center">
<img width="450" alt="image" src="https://github.com/user-attachments/assets/2798d60e-d253-4791-a3e2-ca01bb221c99">
</p>


3. Run cdk synth to synthesize an AWS CloudFormation template from your CDK code. By using L2 constructs, many of the configuration details required by AWS CloudFormation to facilitate the interaction between your Lambda function and REST API are provisioned for you by the AWS CDK.

```s
cdk synth
```

<p align="center">
<img width="450" alt="image" src="https://github.com/user-attachments/assets/aef65cce-3b59-4d7b-b68a-25baa931fd03">
</p>

> NOTE: If successful, you will see an AWS CloudFormation template being constructed and displayed via the Terminal. 

---------

#### 6. Deploy your application 

1. From the root of your project, run the following. Confirm changes if prompted:

```s
cdk bootstrap
```

<p align="center">
<img width="450" alt="image" src="https://github.com/user-attachments/assets/871ab1bb-425f-4065-bad9-7edec69d60a7">
</p>


> ERRORS: If you attempt to run `cdk deploy` prior to running `cdk bootstrap` you will get an error. If you run `cdk bootstrap` after you have previously run the command in this account, you will also get an error. What is happeing is when you run `cdk bootstrap` AWS is attempting to set up some resources that CloudFormation will need to execute your CDK project -- specifically it creates an S3 bucket that it needs to store various config items. This S3 bucket is most commonly the error you receive when running `cdk bootstrap` for first or second time. If you haven't run it once you'll get an error that says you need an S3 bucket. If you have run it before, you'll get an error saying you already have an S3 bucket. Just disposition the error accordingly. 

<p align="center">
<img width="450" alt="image" src="https://github.com/user-attachments/assets/3834daba-8283-4334-89d5-d03bede683b6">
</p>

<p align="center">
<img width="450" alt="image" src="https://github.com/user-attachments/assets/1a56948d-2385-4ae0-821a-816f4e71d5d4">
</p> 

<p align="center">
<img width="450" alt="image" src="https://github.com/user-attachments/assets/8f0c1786-b9be-4129-abbe-d01499392535">
</p>

2. Now you can deploy your stack 

```s
cdk deploy
```

2. When deployment completes, the AWS CDK CLI will output your endpoint URL. Copy this URL for the next step. The following is an example:

<p align="center">
<img width="450" alt="image" src="https://github.com/user-attachments/assets/0416d9a8-7cbc-40ae-959d-6809ade1e3d9">
</p>

<p align="center">
<img width="450" alt="image" src="https://github.com/user-attachments/assets/9bc11815-32bc-4169-8278-30e2cacab28a">
</p>