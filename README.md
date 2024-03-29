# dlq-to-telegram

Sends SNS messages (in particular, dead-letter queue messages from AWS Lambda) to a (private) Telegram chat. (prereq: deploy [this template first](https://github.com/jeshan/lambda-default-dlq) to ensure your functions have dead-letter queue set up)

![Demo](screenshot.png)

If needed, you can create a virtual env with `pipenv install`

This includes a deployment pipeline on AWS. Or deploy the pipeline manually with this button: 

<a href="https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=dlq-to-telegram-deployment-pipeline&templateURL=https://s3.amazonaws.com/jeshan-oss-public-files/dlq-to-telegram-deployment-pipeline-template.yaml">
<img src="https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png"/>
</a>


You will need a telegram bot token and a telegram group (chat) id. Define them in us-east-1 as follows:

```bash
aws ssm put-parameter --name bot-token --type SecureString --value $YOUR_TOKEN --region us-east-1 
aws ssm put-parameter --name /dlq-to-telegram/chat-id --type String --value $YOUR_CHAT_ID --region us-east-1
sceptre launch -y app
``` 

# Adding private sceptre configuration
The build process also generates boilerplate configuration with `python generate-config.py`.

You have the ability to provide sceptre with the necessary configuration and credentials that you will want to keep private.
Read the buildspec for this, in particular:
`aws s3 sync s3://${PRIVATE_BUCKET}/github.com/$REPO/master .`

To generate sceptre configuration for a private environment, you can run something like:
`python generate-config.py production`



You can place your private sceptre configuration at that location in a private bucket and they will be pulled on build.
There's a script available to send these files to S3: Edit your private bucket in `upload-private-config.sh` and run it.
You need to create the role so that your deployment pipeline has permissions to deploy. run `python put-target-deployment-roles.py`
Add the target account number in `config/app/${ENV}/config.yaml`.
Then, run the pipeline. That's all what's needed for sceptre to work.
