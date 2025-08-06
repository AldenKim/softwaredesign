# Automating AWS Resource Management

The Tweeter project uses a variety of AWS resources including Lambda functions, API Gateway endpoints, DynamoDB tables, and SQS queues (~35 resources in all). Creating and configuring these resources manually through the AWS web console is tedious and error-prone. Compounding the problem is that the configurations of these resources will likely change over the course of the project, requiring you to manually re-configure resources multiple times. Fortunately, AWS provides tools to automate creating and configuring resources so you don't have to do it manually.

## AWS CLI and Shell Scripts

One approach to automating these tasks is to use the AWS CLI. The CLI provides commands for creating, updating, and deleting all types of AWS resources. You can write shell scripts that call the CLI to manage your resources. For example, in Tweeter you will have ~16 Lambda functions. Each time you modify your server code you will need to upload the new code to all 16 of your Lambdas. Obviously, doing this manually would be extremely tedious. Alternatively, you could write a shell script that automatically uploads your modified server code to each Lambda. An example of doing this is the [Server Scripts](./server-scripts.md) we provide for uploading Lambda code. You could also write similar scripts for managing API Gateway endpoints, DynamoDB tables, and SQS queues.

## AWS SAM

Another automation strategy is to use "configuration as code". With this approach you create a text file describing your project's AWS resources and their configurations (typically a JSON or YAML file). AWS provides tools such as CloudFormation and Serverless Application Model (SAM) that can take your configuration file and automatically create, update, and delete resources as needed to match the desired configuration. This way, you don't have to write shell scripts. You just need to describe the resources you want in a file and run a tool to "make it so". You can apply your configuration as often as needed as your project evolves. Here is an [example SAM configuration file](./example-template.md). This file creates a DynamoDB table, an SQS queue, a Lambda function connected to an API Gateway endpoint, and a Lambda function connected to the SQS queue. To create and/or update your resources, just run the `sam` CLI passing it your configuration file.

## Tweeter + SAM

With SAM it is possible to do the Tweeter project without ever manually creating or configuring resources. AWS provides a [SAM Tutorial](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-getting-started-hello-world.html) that explains how to install the SAM CLI on your computer and how to add SAM to your project. In the tutorial they use the `sam init` command to initialize a brand new SAM project. In Tweeter the Phase 3 specification tells you to create a `tweeter-server` module within your project. Since you will create the `tweeter-server` module manually, you do not need to run the `sam init` command. Instead, do the following:

1. Create an empty S3 bucket that SAM can use to deploy your project.
1. In the `tweeter-server` directory create a file named `template.yaml` containing [this text](./tweeter-template.md). `template.yaml` demonstrates how to create the types of resources you will need in your Tweeter project. Over the course of the project you will need to modify this file to add your project's resources. In Milestone 3 you will define Lambda functions that are connected to API Gateway endpoints. In Milestone 4A you will define DynamoDB tables. In Milestone 4B you will define SQS queues and Lambda functions that are connected to them.
1. In the `tweeter-server` directory create a file named `samconfig.toml` containing [this text](./tweeter-samconfig.md). `samconfig.toml` contains some properties telling SAM how to deploy the project's resources to AWS. In `samconfig.toml` modify the `s3_bucket` property to be the name of the S3 bucket you created above. Also, modify the `region` property to match the region in which SAM should create resources.
1. In the `tweeter-server` directory create a sub-directory named `layer`. `layer` will contain the code for your project's Lambda layer.
1. Within the `tweeter-server/layer` directory create a sub-directory named `nodejs`.
1. Copy the `tweeter-server/node_modules` directory into the `tweeter-server/layer/nodejs` directory. Be sure to copy the `node_modules` directory itself, not just its contents.
    - Each time you change `tweeter-server`'s dependencies, you will need to re-copy `tweeter-server/node_modules` into `tweeter-server/layer/nodejs` (i.e., the layer needs to be updated whenever the contents of `node_modules` changes).
1. As you develop your server, when you want to re-deploy your server code to AWS, run the following commands in the `tweeter-server` directory:
    1. Build your server code.
        - `npm run build`
        - This will place the generated Javascript files in your project's `dist/` directory.
    1. Prepare your project for deployment to AWS.
        - `sam build`
    1. Deploy your project to AWS.
        - `sam deploy`
        - SAM will automatically create, modify, and delete resources as needed to produce a configuration matching your `template.yaml` file.
1. That's it.
