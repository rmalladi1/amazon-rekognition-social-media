# Introduction to Amazon Rekognition with Social Media Image Moderator

[Amazon Rekognition](https://aws.amazon.com/rekognition/), a deep learning-based service, makes it easy to add image and video analysis
to your applications. Explore Rekognition features and build a Social Media Image Moderator in this walk-through. Begin by exploring Rekognition's face detection and comparison capabilities. Next, use the [Detect Unsafe Images API](https://docs.aws.amazon.com/rekognition/latest/dg/procedure-moderate-images.html) to detect and remove suggestive content as its being uploaded into social media by building an Image Moderator.

This example is based on robperc's Serverless Reference Architecture: Image Moderation Chatbot repository here: https://github.com/aws-samples/lambda-refarch-image-moderation-chatbot

Rekognition can identify images that contain suggestive or explicit content, helping administrators of photo sharing sites, forums, e-commerce platforms and more protect their users. Image moderation provides a hierarchical list of labels for each image with confidence scores to enable fine-grained control over what images to allow. In this example, images found to contain explicit or suggestive content labels above a minimum confidence interval are automatically removed by a Slack chatbot, and a message explaining the removal is posted to the originating channel. The reference architecture presented here uses [Amazon API Gateway](https://aws.amazon.com/api-gateway/), [AWS Lambda](https://aws.amazon.com/lambda/), and [Amazon Rekognition](https://aws.amazon.com/rekognition/)'s [image moderation](https://aws.amazon.com/rekognition/faqs/#image-moderation) to create a [Serverless](https://aws.amazon.com/serverless/#getstarted) workflow for protecting a user's social media channel. This example is intended to work with [Slack](https://slack.com/) using a chatbot, but could also be modified to work with other popular social media or chat apps such as [Facebook Messenger](https://www.messenger.com/).

This repository contains sample code for all the Lambda functions depicted in the diagram below as well as an [AWS CloudFormation](https://aws.amazon.com/cloudformation/) template for creating the functions and related resources.

For further ideas applying Amazon Rekognition features to social media check out [Finding Missing Persons using Social Media and Rekognition](https://github.com/aws-samples/finding-missing-persons-using-social-media-and-amazon-rekognition) and [Image Recognition and Processing Backend Serverless reference architecture](https://github.com/awslabs/lambda-refarch-imagerecognition)

## Rekognition Introduction
1. Sign in to your AWS account.
1. Get familiar with the facial detection features of Rekognition by completing the following tutorial:
https://aws.amazon.com/getting-started/tutorials/detect-analyze-compare-faces-rekognition/
1. While in the Rekognition page of the AWS Management Console, note the additional demos provided. Test out demos on 'Object and scene detection' and 'Text in image' now or at a later time.
1. Choose the Image Moderation demo by clicking on 'Image Moderation' under 'Demos'
1. Analyze the sample images. Note the Response for the swimwear image and the confidence level of over 90%.

## Image Moderator
Now build the Image Moderator. This example uses a Slack chatbot and AWS Lambda to call the Rekognition Detect Unsafe Images API. The architecture is explained below.

![screenshot for instruction](images/Architecture.png)

1. A user posts a message containing an image to a chat app channel that’s monitored by a chatbot.
1. The chat app posts the event to an Amazon API Gateway API for the chatbot.
1. The chatbot validates the event. This event triggers an AWS Lambda function that downloads the image.
1. Amazon Rekognition’s image moderation feature checks the image for suggestive or explicit content.
1. The chat app API deletes an image containing explicit or suggestive content from the chat channel.
1. The chatbot uses the chat app API to post a message to the chat channel detailing deletion of the image.


## Running the Example
### Preparing Slack
First make sure you're logged in to Slack, then follow these instructions to prep your bot:
1. [Create an app](https://api.slack.com/apps?new_app=1) ([Documentation](https://api.slack.com/slack-apps#creating_apps))
1. From the `Basic Information` tab under `Settings` take note of the `Verification Token` as it will be required later
1. Navigate to the `OAuth & Permissions` tab under `Features`
1. Under the `Permissions Scopes` section add the following permission scopes
    * channels:history
    * chat:write:bot
    * files:read
    * files:write:user
1. Click `Save Changes`
1. Click `Install App to Team` then `Authorize` then note the `OAuth Access Token` as it will be required later

### Launching the Bot Backend on AWS

#### Option 1: Launch from Serverless Application Repository
This bot can be launched into any region that supports the underlying services from the [Serverless Application Repository](https://aws.amazon.com/serverless/serverlessrepo/) using the instructions below:

1. Navigate to the [application details page](https://serverlessrepo.aws.amazon.com/applications/arn:aws:serverlessrepo:us-east-1:426111819794:applications~image-moderation-chatbot) for the chatbot.
1. Click `Deploy`
1. From the region dropdown in the top right ensure you have the desired region to deploy into selected
1. Input the appropriate application parameters under `Configure application parameters`
1. Scroll to the bottom of the page and click `Deploy` to deploy the chatbot

#### Option 2: Launch the CloudFormation Template Manually
If you would like to deploy the template manually, you need a S3 bucket in the target region, and then package the Lambda functions into that S3 bucket by using the `aws cloudformation package` utility.

Set environment variables for later commands to use:

```bash
S3BUCKET=[REPLACE_WITH_YOUR_BUCKET]
REGION=[REPLACE_WITH_YOUR_REGION]
STACKNAME=[REPLACE_WITH_DESIRED_NAME]
VTOKEN=[REPLACE_WITH_VERIFICATION_TOKEN]
ATOKEN=[REPLACE_WITH_OAUTH_ACCESS_TOKEN]
```

Then go to the `cloudformation` folder and use the `aws cloudformation package` utility

```bash
cd cloudformation

aws cloudformation package --region $REGION --s3-bucket $S3BUCKET --template image_moderator.serverless.yaml --output-template-file image_moderator.output.yaml
```
Last, deploy the stack with the resulting yaml (`image_moderator.output.yaml`) through the CloudFormation Console or command line:

```bash
aws cloudformation deploy --region $REGION --template-file image_moderator.output.yaml --stack-name $STACKNAME --capabilities CAPABILITY_NAMED_IAM --parameter-overrides VerificationToken=$VTOKEN AccessToken=$ATOKEN
```

### Finalize Slack Event Subscription
1. Navigate to the created stack in the CloudFormation console and note the value for the `RequestURL` output from the created stack as it will be required later
1. Return to the Slack app settings page for the Slack app created earlier
1. Navigate to the `Event Subscriptions` tab under `Features` and enable events
1. In the `Request URL` field enter the `RequestURL` value noted earlier
1. Click `Add Workspace Event` and select `message.channels`
1. Click `Save Changes`


## Testing the Example
To test the example open your Slack bot and attempt to upload the sample images from the Amazon Rekognition console demo, which can be downloaded from the links below:
- [Family Picnic](https://dhei5unw3vrsx.cloudfront.net/images/family_picnic_resized.jpg) (will not be removed by bot)
- [Yoga Swimwear](https://dhei5unw3vrsx.cloudfront.net/images/yoga_swimwear_resized.jpg) (will be removed by bot)

![testing of example gif](images/TestingExample.gif)

## Exploring Results
From the AWS Management Console services page, choose Lambda. Ensure you are in the correct Region using the dropdown menu in the upper right. Choose the function beginning with the name "ImageModerationChatbot-ImageModeratorFunction-". Here you can view the function that evaluates your Slack images. To check out the function output, choose 'Monitoring' then 'View logs in CloudWatch' and choose latest Log Stream.

Return to the function page and scroll down to 'Environment Variables'. Note that the minimum confidence level for the Image Moderator has been defined as an Environment Variable with a default value of 80%. This can be edited in the SAM template (for tracking changes), or for testing purposes you can adjust it here. Highlight the value, change it, and Click 'Save' to Save the function. Now upload various images to see how confidence value affects their removal. For example, the following image of a runner is removed at 50% confidence, but not at the 80% default value.
- [Runner](https://images.unsplash.com/photo-1461896836934-ffe607ba8211?ixlib=rb-0.3.5&ixid=eyJhcHBfaWQiOjEyMDd9&s=2c33a9425efd786461261c7a92d22634&auto=format&fit=crop&w=500&q=60)

## Cleaning Up the Stack Resources

To remove all resources created by this example, do the following:

1. Delete the CloudFormation stack.
1. Delete the CloudWatch log groups associated with each Lambda function created by the CloudFormation stack.

## CloudFormation Template Resources
The following sections explain all of the resources created by the CloudFormation template provided with this example.

### AWS Lambda
- **ImageModeratorFunction** - Lambda function that validates incoming Slack event messages, checks them for images containing explicit content, and orchestrates the removal of images found to contain explicit content from Slack.
- **ImageModeratorFunctionImageModeratorAPIPostPermissionTest** - Implicitly created Lambda permission, allows API Gateway Test stage to call Lambda function.
- **ImageModeratorFunctionImageModeratorAPIPostPermissionProd** - Implicitly created Lambda permission, allows API Gateway Prod stage to call Lambda function.

### AWS IAM
- **ImageModeratorFunctionRole** - Implicitly created IAM Role with policy that allows Lambda function to invoke "rekognition:DetectLabels" and "rekognition:DetectModerationLabels" API calls and write log messages to CloudWatch Logs.

### Amazon API Gateway
- **ImageModeratorAPI:** - API for image moderation chatbot
- **ImageModeratorAPIProdStage** - Implicitly created production stage for API
- **ImageModeratorAPIDeploymentXXXXXXXXX** - Implicitly created deployment for production stage of API


## License

This reference architecture sample is licensed under Apache 2.0.
