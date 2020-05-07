# T3chFlicks - API with ALB and Lambda
An example of an API created using 
AWS Lambda and Application Load Balancer. 
See the accompanying blog post [here](https://medium.com/@t3chflicks/cheaper-than-api-gateway-alb-with-lambda-using-cloudformation-b32b126bbddc).

### Usage
```
$ curl loadb-LoadB-R7RVQD09YC9O-1401336014.eu-west-1.elb.amazonaws.com
Hello World!
```

### Architecture
![Architecture](./architecture.png)

### Setup
1. Deploy VPC
    * `aws cloudformation create-stack --stack-name vpc --template-body file://aws/vpc.yml --capabilities CAPABILITY_NAMED_IAM`
    * tutorial for VPC can be found [here](https://medium.com/@t3chflicks/virtual-private-cloud-on-aws-quickstart-with-cloudformation-4583109b2433)
1. Deploy Service
    * `aws cloudformation create-stack --stack-name service --template-body file://aws/service.yml --capabilities CAPABILITY_NAMED_IAM`
1. Deploy Service with CORS enabled
    * `aws cloudformation update-stack --stack-name service --template-body file://aws/service_CORS.yml --capabilities CAPABILITY_NAMED_IAM`

---

This project was created by [T3chFlicks](https://t3chflicks.org) a tech focused education and service company.

---


