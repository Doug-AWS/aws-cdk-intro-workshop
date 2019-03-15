+++
title = "API Gateway"
weight = 300
+++

Next step is to add an API Gateway in front of your function. API Gateway will
expose a public HTTP endpoint that anyone on the internet can hit with an HTTP
client such as [curl](https://curl.haxx.se/) or a web browser.

We will use [Lambda proxy
integration](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-create-api-as-simple-proxy-for-lambda.html)
mounted to the root of the API. This means that any request to any URL path will
be proxied directly to your Lambda function, and the response from the function
will be returned back to the user.

## Install the API Gateway construct library

```console
npm install @aws-cdk/aws-apigateway@0.25.3
```

{{% notice info %}}

**Windows users**: on Windows, you will have to stop the `npm run watch` command
that is running in the background, then run `npm install`, then start
`npm run watch` again. Otherwise you will get an error about files being
in use.

{{% /notice %}}

## Add a LambdaRestApi construct to your stack

Let's define an API endpoint and associate it with your Lambda function.
Add the following highlighted code to *cdk-workshop-stack.ts*:

{{<highlight ts "hl_lines=3 15-18">}}
import cdk = require('@aws-cdk/cdk');
import lambda = require('@aws-cdk/aws-lambda');
import apigw = require('@aws-cdk/aws-apigateway');

export class CdkWorkshopStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const hello = new lambda.Function(this, 'HelloHandler', {
      runtime: lambda.Runtime.NodeJS810,
      code: lambda.Code.asset('lambda'),
      handler: 'hello.handler'
    });

    // defines an API Gateway REST API resource backed by your "hello" function.
    new apigw.LambdaRestApi(this, 'Endpoint', {
      handler: hello
    });
  }
}
{{</highlight>}}

That's it. This is all you need to do in order to define an API Gateway which
proxies all requests to an AWS Lambda function.

## cdk diff

Let's see what's going to happen when we deploy this:

```console
cdk diff
```

Output should look like this:

```
Resources
[+] AWS::Lambda::Permission HelloHandler/ApiPermission.ANY.. HelloHandlerApiPermissionANYAC4E141E
[+] AWS::Lambda::Permission HelloHandler/ApiPermission.Test.ANY.. HelloHandlerApiPermissionTestANYDDD56D72
[+] AWS::Lambda::Permission HelloHandler/ApiPermission.ANY..{proxy+} HelloHandlerApiPermissionANYproxy90E90CD6
[+] AWS::Lambda::Permission HelloHandler/ApiPermission.Test.ANY..{proxy+} HelloHandlerApiPermissionTestANYproxy9803526C
[+] AWS::ApiGateway::RestApi Endpoint EndpointEEF1FD8F
[+] AWS::ApiGateway::Deployment Endpoint/Deployment EndpointDeployment318525DA37c0e38727e25b4317827bf43e918fbf
[+] AWS::ApiGateway::Stage Endpoint/DeploymentStage.prod EndpointDeploymentStageprodB78BEEA0
[+] AWS::IAM::Role Endpoint/CloudWatchRole EndpointCloudWatchRoleC3C64E0F
[+] AWS::ApiGateway::Account Endpoint/Account EndpointAccountB8304247
[+] AWS::ApiGateway::Resource Endpoint/Default/{proxy+} Endpointproxy39E2174E
[+] AWS::ApiGateway::Method Endpoint/Default/{proxy+}/ANY EndpointproxyANYC09721C5
[+] AWS::ApiGateway::Method Endpoint/Default/ANY EndpointANY485C938B
```

That's nice. This one line of code added 12 new resources to your stack.

## cdk deploy

Okay, ready to deploy?

```console
cdk deploy
```

## Stack outputs

When deployment is complete, you'll notice this line:

```
CdkWorkshopStack.Endpoint8024A810 = https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/
```

This is a [stack output](https://awslabs.github.io/aws-cdk/cloudformation.html#outputs) that's
automatically added by the API Gateway construct and includes the URL of the API Gateway endpoint.

## Testing your app

Let's try to hit this endpoint with `curl`. Copy the URL and execute (your
prefix and region will likely be different).

{{% notice info %}}
If you don't have [curl](https://curl.haxx.se/) installed, you can always use
your favorite web browser to hit this URL.
{{% /notice %}}

```console
curl https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/
```

Output should look like this:

```
Hello, CDK! You've hit /
```

Obviously, you can also use your web browser for this:

![](./browser.png)

If this is the output you received, your app works!

## What if it didn't work?

If you received a 5xx error from API Gateway, it is likely one of two issues:

1. The response your function returned is not what API Gateway expects. Go back
   and make sure your handler returns a response that includes a `statusCode`,
   `body` and `header` fields (see [Write handler runtime
   code](./200-lambda.html)).
2. Your function failed for some reason. To debug this, you can quickly jump to [this section](../40-hit-counter/500-logs.html)
   to learn how to view your Lambda logs.

---

Good job! In the next chapter, we'll write your own reusable construct.
