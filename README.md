# Create a CI/CD Pipeline for an E-commerce Site (Magento)

## Deployment Instructions

1. Sign in to [Magento Marketplace](https://marketplace.magento.com/). Create an account if you don't have one.
1. Create Magento access keys. Keep the browser tab open. You'll need the values later.
![Magento Marketplace Screen 1](/mp1.png "Magento Marketplace Screen 1")

![Magento Marketplace Screen 2](/mp2.png "Magento Marketplace Screen 2")

![Magento Marketplace Screen 3](/mp3.png "Magento Marketplace Screen 3")

![Magento Marketplace Screen 4](/mp4.png "Magento Marketplace Screen 4")

1. Sign-in to your [AWS console](https://console.aws.amazon.com/).
1. Transfer to Tokyo region.
1. Go to [Secrets Manager](https://ap-northeast-1.console.aws.amazon.com/secretsmanager/home).
1. Store a new secret. Choose "Other type of secrets".
1. Add 2 key-value pairs - one for `username` and another for `password`. Use Magento Marketplace Public Key as the value for `username` and the Private Key for `password`.
1. Provide a secret name. Remember it as you'll need that later.
1. Keep clicking on "Next" - leave all settings at default. At the end, click "Store".
1. Store another secret. Also choose "Other type of secrets".
1. Add 6 key-value pairs - `username`, `password`, `firstName`, `lastName`, `email`, `urlPath`. These will be the values used for your Magento admin. For `urlPath`, your admin page will be located in that path. So just use any acceptable URL path string.
1. Provide a secret name. Remember it as you'll need that later.
1. Keep clicking on "Next" - leave all settings at default. At the end, click "Store".
1. Go to [Cloud9](https://ap-northeast-1.console.aws.amazon.com/cloud9/home/product).
1. Create an environment. You need to name it uniquely if you are sharing an account with others.
1. Keep clicking on "Next Step" - leave all settings at default. At the end, click "Create Environment".
1. The creation should take a min or two. Meanwhile, you can proceed to the next steps. Keep the browser tab open. You'll need the environment later.
1. Go to [CodeCommit](https://ap-northeast-1.console.aws.amazon.com/codesuite/codecommit/start).
1. Create a repository. You need to name it uniquely if you are sharing an account with others.
1. Keep the browser tab open. You'll need to copy the repository URL later.
1. Go to [ECR](https://ap-northeast-1.console.aws.amazon.com/ecr/get-started).
1. Get started. Create a private repository. You need to name it uniquely if you are sharing an account with others.
1. Keep the browser tab open. You'll need to copy the repository URL later.
1. Go back to your Cloud9 environment. Run the following commands from the terminal:
```
git clone https://github.com/engr-lynx/aws-mage-infra.git
cd aws-mage-infra
. ./bin/env.sh
./bin/prep.sh
```
1. Run the following commands from the terminal:
```
./bin/deploy.sh
```
1. Open another terminal in your Cloud9 environment. Run the following commands:
```
docker pull public.ecr.aws/z0z6r0u2/php7.4-apache:latest
docker tag public.ecr.aws/z0z6r0u2/php7.4-apache:latest <your ECR repo>:latest
aws ecr get-login-password --region ap-northeast-1 | docker login --username AWS --password-stdin <AWS account ID>.dkr.ecr.ap-northeast-1.amazonaws.com
docker push <your ECR repo>:latest
```
1. Go to [App Runner](https://ap-northeast-1.console.aws.amazon.com/apprunner/home).
1. Create a service.
1. Provide your ECR repository URI with the latest tag in "Container image URI".
1. Select Automatic Deployment.
1. Click Next.
1. Provide a service name. You need to name it uniquely if you are sharing an account with others.
1. Replace the port number to 80.
1. Click Next.
1. Click Create & Deploy.
1. Go to [CodePipeline](https://ap-northeast-1.console.aws.amazon.com/codesuite/codepipeline/start)
1. Create pipeline.
1. Provide a pipeline name. You need to name it uniquely if you are sharing an account with others.
1. Click Next.
1. Select AWS CodeCommit as service provider.
1. Select the CodeCommit repository you created.
1. Put in "master" as the branch.
1. Click Next.
1. Select AWS CodeBuild as build provider.
1. Click "Create Project". This will open a new window.
1. Provide a project name. You need to name it uniquely if you are sharing an account with others.
1. Select "Amazon Linux 2" for "Operating System".
1. Select "Standard" for "Runtime".
1. Select "aws/codebuild/amazonlinux2-x86_64-standard:3.0" for "Image".
1. Select "Insert build commands".
1. Go to [RDS](https://ap-northeast-1.console.aws.amazon.com/rds/home). Get the created DB endpoint name.
1. Go to [ElasticSearch](https://ap-northeast-1.console.aws.amazon.com/esv3/home). Get the created ES domain endpoint.
1. Go to Secrets Manager. Get the 4 secret names (2 you created, 2 created by CDK).
1. Put in the following build spec:
```
version: "0.2"
env:
  variables:
    BASE_URL: <App Runner default domain>
    DEPLOY_SAMPLE: "true"
    DB_HOST: <DB host name>
    DB_NAME: magento
    ES_HOST: https://<ES host domain>
    DOCKER_BUILDKIT: 1
  secrets-manager:
    DB_USERNAME: <DB Credential Secret>:username
    DB_PASSWORD: <DB Credential Secret>:password
    ES_USERNAME: <ES Credential Secret>:username
    ES_PASSWORD: <ES Credential Secret>:password
    MP_USERNAME: <Magento Marketplace Credential Secret>:username
    MP_PASSWORD: <Magento Marketplace Credential Secret>:password
    ADMIN_FIRST_NAME: <Magento Admin Secret>:firstName
    ADMIN_LAST_NAME: <Magento Admin Secret>:lastName
    ADMIN_EMAIL: <Magento Admin Secret>:email
    ADMIN_URL_PATH: <Magento Admin Secret>:urlPath
    ADMIN_USERNAME: <Magento Admin Secret>:username
    ADMIN_PASSWORD: <Magento Admin Secret>:password
phases:
  install:
    runtime-versions:
      docker: 19
    commands: []
  pre_build:
    commands:
      - aws ecr get-login-password | docker login --username AWS --password-stdin <AWS account ID>.dkr.ecr.ap-northeast-1.amazonaws.com
      - docker pull <ECR repository>:latest || true
  build:
    commands: docker build --build-arg BUILDKIT_INLINE_CACHE=1 --build-arg BASE_URL="${BASE_URL}" --build-arg DEPLOY_SAMPLE="${DEPLOY_SAMPLE}" --build-arg DB_HOST="${DB_HOST}" --build-arg DB_NAME="${DB_NAME}" --build-arg ES_HOST="${ES_HOST}" --build-arg DB_USERNAME="${DB_USERNAME}" --build-arg DB_PASSWORD="${DB_PASSWORD}" --build-arg ES_USERNAME="${ES_USERNAME}" --build-arg ES_PASSWORD="${ES_PASSWORD}" --build-arg MP_USERNAME="${MP_USERNAME}" --build-arg MP_PASSWORD="${MP_PASSWORD}" --build-arg ADMIN_FIRST_NAME="${ADMIN_FIRST_NAME}" --build-arg ADMIN_LAST_NAME="${ADMIN_LAST_NAME}" --build-arg ADMIN_EMAIL="${ADMIN_EMAIL}" --build-arg ADMIN_URL_PATH="${ADMIN_URL_PATH}" --build-arg ADMIN_USERNAME="${ADMIN_USERNAME}" --build-arg ADMIN_PASSWORD="${ADMIN_PASSWORD}" --cache-from 152242201060.dkr.ecr.ap-northeast-1.amazonaws.com/magento-webimagerepo43565df4-j1hrivpjeztr:latest -t 152242201060.dkr.ecr.ap-northeast-1.amazonaws.com/magento-webimagerepo43565df4-j1hrivpjeztr:latest .
  post_build:
    commands:
      - docker push <ECR repository>
```
1. Click "Switch to Editor".
1. Click "Continue to Pipeline". This will bring you back to the main CodePipeline window.
1. Click Next.
1. Click "Skip Deploy Stage".
1. Click "Create pipeline".
1. Go to your Cloud9 enviroment. Open a new terminal. Run the following commands.
```
git clone https://github.com/engr-lynx/mage2-cont.git
cd mage2-cont/
git remote add aws <CodeCommit https URL>
git push aws master
```

## Clean-up Instructions

1. Sign-in to your [AWS console](https://console.aws.amazon.com/).
1. Transfer to Tokyo region.
1. Go to [CodePipeline](https://ap-northeast-1.console.aws.amazon.com/codesuite/codepipeline/start)
1. Delete the pipeline you created.
1. Go to [App Runner](https://ap-northeast-1.console.aws.amazon.com/apprunner/home).
1. Delete the service you created.
1. Go to [ECR](https://ap-northeast-1.console.aws.amazon.com/ecr/get-started).
1. Delete the repository you created.
1. Go to [CodeCommit](https://ap-northeast-1.console.aws.amazon.com/codesuite/codecommit/start).
1. Delete the repository you created.
1. Go to [CloudFormation](https://ap-northeast-1.console.aws.amazon.com/cloudformation/home?region=ap-northeast-1).
1. Delete the Magento cloudformation stack that was created by CDK.
1. Go to [Cloud9](https://ap-northeast-1.console.aws.amazon.com/cloud9/home/product).
1. Delete the environment you created.
1. Go to [Secrets Manager](https://ap-northeast-1.console.aws.amazon.com/secretsmanager/home).
1. Delete the 2 secrets you created.
