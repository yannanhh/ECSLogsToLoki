# ECS - CW - KDS - Promtail Lambda - Loki
 Promtail Lambda code: https://github.com/grafana/loki/tree/main/tools/lambda-promtail

## Upload `lambda-promtail` image to ECR


```sh
aws ecr get-login-password \
--profile USWest1 \
--region us-west-1 | docker login \
--username AWS \
--password-stdin 093575270853.dkr.ecr.us-west-1.amazonaws.com

docker pull public.ecr.aws/grafana/lambda-promtail:main
docker tag public.ecr.aws/grafana/lambda-promtail:main 093575270853.dkr.ecr.us-west-1.amazonaws.com/demo-grafana-loki-ecr:latest

docker push 093575270853.dkr.ecr.us-west-1.amazonaws.com/demo-grafana-loki-ecr:latest
```

## Grant ECR permissions to Lambda

demo-grafana-loki-ecr

https://docs.aws.amazon.com/lambda/latest/dg/images-create.html#configuration-images-permissions

```json
{
	"Sid": "LambdaECRImageRetrievalPolicy",
	"Effect": "Allow",
	"Principal": {
		"Service": "lambda.amazonaws.com"
	},
	"Action": [
	  	"ecr:BatchGetImage",
	  	"ecr:GetDownloadUrlForLayer"
	]
}   
```

## Create ECS Task 

Task definition:
  
```json
{
  "name": "demo-grafana-loki-cw-sample-app-task",
  "command": [
    "/bin/sh -c \"while true; do sleep 60; echo hello_from_sample_app; done\""
  ],
  "entryPoint": ["sh","-c"],
  "essential": true,
  "image": "alpine:3.12",
  "logConfiguration": {
    "logDriver": "awslogs"
  }
}
```

## Kinesis

### Mac install terraform:

```sh
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

### Lambda Promtail and Kinesis

add aws provider
```tf
provider "aws" {
  region                  = "us-west-1"
  profile                 = "USWest1"
}
```

change main.tf lambda arch to arm64
```tf
resource "aws_lambda_function" "this" {
  architectures = ["arm64"]
  ... 
}
```
init terraform and install aws provider

```sh
terraform init
```



### Connect KDS to Promtail Lambda, Loki
```sh
terraform apply \
-var "name=demo-grafana-loki-cw-kds-stack" \
-var "lambda_promtail_image=093575270853.dkr.ecr.us-west-1.amazonaws.com/demo-grafana-loki-ecr:latest" \
-var "write_address=http://ec2-54-241-127-143.us-west-1.compute.amazonaws.com:3100/loki/api/v1/push" \
-var "password=admin" \
-var "username=admin" \
-var 'kinesis_stream_name=["demo-grafana-loki-cw-kds"]' \
-var 'extra_labels=job,cwkds' \
-var 'omit_extra_labels_prefix=true' \
-var 'skip_tls_verify="true"' \
-var 'keep_stream="true"'
```

## View logs in Grafana UI

![CW-KDS-Loki-Grafana-UI](example-cw-kds-lambda.png)
