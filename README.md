## tkim

- 2023/11/12 tkim
- 이 프로젝트는 laravel 10 프로젝트를 aws lambda로 실행해 보기 위한 poc이다.
- 수순은 https://ma-vericks.com/blog/serverless-laravel-app/ 을 참고하였다.
- 환경은 Windows VDI,

- 이 프로젝트는 laravel-20231112-poc-bref 에 -cloudfront-s3 를 추가한다.
- https://bref.sh/docs/use-cases/websites
- serverless설치에 대한 설명은 `gh gist view 7d4760fcd9646f1b57184a3f73c3f273 -w` 를 참고한다.


### Project 만들기 (compact version)
```cmd
cd /temp
laravel new laravel-20231112-poc-bref-cloudfront-s3
cd laravel-20231112-poc-bref-cloudfront-s3
git init
gh repo create --public laravel-20231112-poc-bref-cloudfront-s3 --source=.
composer require bref/bref bref/laravel-bridge --update-with-dependencies
php artisan vendor:publish --tag=serverless-config
cat serverless.yml # change ap-east-1 to ap-northeast-1
php artisan config:clear
serverless deploy
serverless bref:cli --args="--version"
serverless remove
```

## how to set environment variables
```cmd
setx AWS_ACCESS_KEY_ID AKIATHGXHC22YXXCGYIX /m
setx AWS_SECRET_ACCESS_KEY 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 /m
```

## prerequisite
- install serverless
    - `npm install -g serverless`
    - `serverless --version`

## deploy steps
```cmd
serverless config credentials --provider aws --key %AWS_ACCESS_KEY_ID% --secret %AWS_SECRET_ACCESS_KEY%
composer require bref/bref bref/laravel-bridge --update-with-dependencies
php artisan vendor:publish --tag=serverless-config
cat serverless.yml # change ap-east-1 to ap-northeast-1
php artisan config:clear
serverless deploy
serverless deploy list
```

## deploy check
- Lambda Functions: `laravel-dev-web` is for web and `laravel-dev-artisan` is for artisan.
- Runtime: `Custom runtime on Amazon Linux 2`
- API Gateway: endpoint `laravel-dev-web` exists

## log
```cmd
serverless logs --function web
```

## serverless-lift
- install serverless-lift, and upload /public to s3 bucket by setting serverless.yml
- it is need to install serverless-lift plugin each project.
```cmd
serverless plugin install -n serverless-lift
serverless lift --stage dev --region ap-northeast-1 --verbose
serverless deploy --stage dev --region ap-northeast-1 --verbose
aws cloudfront list-distributions --output yaml
aws cloudformation list-stacks --output yaml
serverless remove --verbose
aws cloudformation delete-stack --stack-name laravel-dev-websiteassets2a73bb69-3fz81fzx26f2
```

## serverless-lift deploy check
- CloudFront:
- CloudFormation: laravel-dev

to see the aws cli result as table,
```cmd
aws cloudformation list-stacks --query "StackSummaries[?StackStatus=='CREATE_COMPLETE'].{StackName:StackName,StackStatus:StackStatus,CreationTime:CreationTime,LastUpdatedTime:LastUpdatedTime,StackId:StackId}" --output table
```

## check s3 bucket
```cmd
aws s3 ls s3://laravel-dev-websiteassets2a73bb69-3fz81fzx26f2
```

## logging strategy?
- logging store should be cloudwatch.


## cloudfront의 에러를 추적하는 방법

- ```
  403 ERROR
  Generated by cloudfront (CloudFront)
  Request ID: QFmiGlIsD0_ndWLn-lPtHKFaqIpEGoL0u-rUKTvi3t13OO45xg-uUg==
  ```
- 위와 같은 에러가 발생하면, cloudfront의 로그를 확인해야 한다.
- cloudfront의 로그는 s3에 저장되어 있다.
- cli 명령으로는 다음과 같다.
- ```
  aws s3 ls 
  aws s3 ls s3://laravel-dev-websiteassets2a73bb69-3fz81fzx26f2/logs/
  ```
  
## powershell 로 확인하는 방법
```pwsh
Import-Module AWSPowerShell.NetCore
Get-AWSPowerShellVersion -ListServiceVersionInfo
```

