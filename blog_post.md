# Cheaper than API Gateway — ALB with Lambda using CloudFormation

An alternative to API gateway is Application Load Balancer. ALB can be connected with Lambda to produce a highly performant, cost effective API. In this article, I demonstrate how to create an API using CloudFormation and Python.

### API Gateway vs. Application Load Balancer

[API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html) and [ALB](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) are two **different** AWS services, however they can both be used to achieve the same thing: send network requests for a service to the service.

API Gateway is pay per request whereas ALBs have an hourly rate, therefore deciding which to use depends on traffic volume. For an in-depth price and feature comparison, I recommend this [article](https://serverless-training.com/articles/save-money-by-replacing-api-gateway-with-application-load-balancer/), which shares this shocking statistic:
> **I expect to pay around $166 per month for ALB, whereas I’m paying $4,163 per month for the exact same service from API Gateway**.

This is a massive saving! That said, the cost difference isn’t unmerited and you do get extra features from API gateway. Dougal Ballantyne, the Head of Product for Amazon API Gateway tweeted:
> **If you are building an API and want to leverage AuthN/Z, request validation, rate limiting, SDK generation, direct AWS service backend, use [#APIGateway](https://twitter.com/hashtag/APIGateway?src=hash). If you want to add Lambda to an existing web app behind ALB you can now just add it to the needed route**

Well, that’s exactly what we’re gonna do!

## API with ALB and Lambda

I am going to build the following the system:

![Architecture Diagram](https://cdn-images-1.medium.com/max/2000/1*y7xiCXLyQi7iZ_nxQUorxg.png)*

Architecture Diagram*

The complete CloudFormation templates can be found [here](https://github.com/sk-t3ch/AWS-API-With-ALB-And-Lambda), split into two templates:

* `vpc.yml` — Configures the VPC. For simplicity, the VPC only contains two public subnets. You can read about a more complex VPC in our [previous article](https://medium.com/@t3chflicks/virtual-private-cloud-on-aws-quickstart-with-cloudformation-4583109b2433).

* `service.yml`— Configures both the ALB and Lambda function.

### Lambda

To create a Lambda using CloudFormation, it is necessary to define a Lambda Function, a Role to run the Lambda and a Permission for the ALB to invoke the Lambda:



### Application Load Balancer

The Load Balancer is placed across public subnets as it needs to be accessible **from** the internet. The ALB is configured to listen to HTTP traffic on port 80 and forward it to the Lambda:

    Lambda:
        Type: AWS::Lambda::Function
        Properties:
        Code:
            ZipFile: |
            def handler(event, context):
            response = {
                'isBase64Encoded': False,
                'statusCode': 200,
                'body': 'HELLO WORLD!'
            }
            return response
        Handler: lambda_function.handler
        Role: !GetAtt LambdaRole.Arn
        Runtime: python3.7
        
    LambdaRole:
        Type: AWS::IAM::Role
        Properties:
        AssumeRolePolicyDocument:
            Statement:
            - Action: ['sts:AssumeRole']
                Effect: Allow
                Principal:
                Service: ['lambda.amazonaws.com']
        Path: /
        ManagedPolicyArns:
            - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

    LambdaFunctionPermission:
        Type: AWS::Lambda::Permission
        Properties:
        FunctionName: !GetAtt Lambda.Arn
        Action: 'lambda:InvokeFunction'
        Principal: elasticloadbalancing.amazonaws.com



The complete code can be accessed [here](https://github.com/sk-t3ch/AWS-API-With-ALB-And-Lambda-Quickstart) and can be deployed using the AWS CLI:

    aws cloudformation create-stack --stack-name service --template-body file://template.yml --capabilities CAPABILITY_NAMED_IAM

After a successful deployment, the DNS name of the ALB can be found in the EC2 section of the AWS console. It should look something like:

    loadb-LoadB-R7RVQD09YC9O-1401336014.eu-west-1.elb.amazonaws.com

Now it is possible to make a request to this URL and get a response:

    $ curl loadb-LoadB-R7RVQD09YC9O-1401336014.eu-west-1.elb.amazonaws.com'

    HELLO WORLD!

![Photo by [Nghia Le](https://unsplash.com/@lephunghia?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/9504/0*2xL0e82Smd6by2-d)*

Photo by [Nghia Le](https://unsplash.com/@lephunghia?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

### Cross Origin Response Sharing

In order to access this service from a browser webpage on a different domain, CORS must be enabled. This is done by setting headers:

    Lambda:
        Type: AWS::Lambda::Function
        Properties:
        Code:
            ZipFile: |
            def handler(event, context):
            response = {
                'isBase64Encoded': False,
                'statusCode': 200,
                'body': 'HELLO WORLD!',
                'headers': {
                    'access-control-allow-methods': 'GET',
                    'access-control-allow-origin': '*',
                    'access-control-allow-headers': 'Content-Type, Access-Control-Allow-Headers'
                }
            }
            return response
        Handler: lambda_function.handler
        Role: !GetAtt LambdaRole.Arn
        Runtime: python3.7


### Use a Domain Name

AWS provides an ugly Load Balancer address such as:

    loadb-LoadB-R7RVQD09YC9O-1401336014.eu-west-1.elb.amazonaws.com

But it’s quite simple to use a custom domain using AWS. Firstly, transfer your DNS management to [Route 53](https://aws.amazon.com/route53/) and then create a new record set aliased to the load balancer.

### Thanks For Reading

I hope you have enjoyed this article. If you like the style, check out [T3chFlicks.org](https://t3chflicks.org/Projects/aws-quickstart-series) for more tech focused educational content ([YouTube](https://www.youtube.com/channel/UC0eSD-tdiJMI5GQTkMmZ-6w), [Instagram](https://www.instagram.com/t3chflicks/), [Facebook](https://www.facebook.com/t3chflicks), [Twitter](https://twitter.com/t3chflicks)).



Resources:

* [https://serverless-training.com/articles/save-money-by-replacing-api-gateway-with-application-load-balancer/](https://serverless-training.com/articles/save-money-by-replacing-api-gateway-with-application-load-balancer/)

* [https://docs.aws.amazon.com/lambda/latest/dg/python-package.html](https://docs.aws.amazon.com/lambda/latest/dg/python-package.html)

* [https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)

* [https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html)