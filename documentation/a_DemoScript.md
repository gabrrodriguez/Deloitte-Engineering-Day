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

#### 3. Define your API Gateway Construct

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

