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
./bin/deploy.sh
```
1. Open another terminal in your Cloud9 environment. Run the following commands:
```
docker pull public.ecr.aws/z0z6r0u2/php7.4-apache:latest
docker tag public.ecr.aws/z0z6r0u2/php7.4-apache:latest <your ECR repo>:latest
docker push <your ECR repo>:latest
```
1. Important: Don't forget to clean-up your deployment.

## Clean-up Instructions

1. Sign-in to your [AWS console](https://console.aws.amazon.com/).
1. Transfer to Tokyo region.
1. Go to [CloudFormation](https://ap-northeast-1.console.aws.amazon.com/cloudformation/home?region=ap-northeast-1).
1. Delete all the CloudFormation stacks you deployed there.
