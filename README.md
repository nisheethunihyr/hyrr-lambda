1. Create 3 buckets: source bucket, destination bucket, library bucket
2. In library bucket upload dependencies (ffmpeglib.zip, ffprobelib.zip, soxlib.zip)
3. Create IAM role with policy -
{

    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogStream",
                "logs:CreateLogGroup",
                "logs:PutLogEvents"
            ],
            "Resource": "*"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject"
            ],
            "Resource": "<source bucket arn>/*"
        },
        {
            "Sid": "VisualEditor2",
            "Effect": "Allow",
            "Action": "s3:PutObject",
            "Resource": "<destination bucket arn>/*"
        }
    ]
}

4. Create one SNS topic.
5. Create cloudFront distribution to serve HTTPS requests for destination s3 bucket and add following policy to destination s3 bucket
{

    "Version": "2012-10-17",
    "Id": "Policy1622789279962",
    "Statement": [
        {
            "Sid": "Stmt1622789277811",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity <CloudFront Distribution Origin Access Identity>"
            },
            "Action": "s3:GetObject",
            "Resource": "<destination bucket arn>/*"
        }
    ]
}

6. Create one Lambda function with runtime env Python 3.7 and add the IAM role created above in step 3
7. Add the code in the lambda function, given in the attached file "LambdaFunction" and in General configuration increase the timeout time to 10 min.
8. Add following environment variables in function's configuration-

ACCESS_KEY <aws account access key>

AUTH <same key which we used in backend as lambda api key>

DESTINATION <destination bucket name>

SECRET_KEY <aws account secret key>

SERVICE_URL <our backend service url>

SOURCE <source bucket name>

DOMAIN_NAME <cloudFrond distribution domain name>

9. Add layer to Lambda function using ARN of above three dependencies(sox, ffmpeg and ffprobe).
10. Add an SNS trigger in our lambda function by giving the name of the topic which we have created in step 9, And check this lambda function should be visible as a subscription to that topic.
11. Now, for sentry implementation create a new project - AWS LAMBDA (PYTHON)
click on button "Add Installation"
Now, new window will open - "Add Sentry's CloudFormation", here click on "go to aws"

Now, "Quick create stack" page will open. Here, add stack name and select the check box - "I acknowledge that AWS CloudFormation might create IAM resources with custom names" and click on "create stack".

After stack creation one form will open where we need to add our aws account Id, stack id should already present then submit these details.

After that screen will open which asks for the function for which we want to set up this sentry project, select our lambda function here and click done.

Now, sentry is configured for our lambda function
You can confirm this by going to lambda function. Here, you can see the layer SentryPythonServerlessSDK is added in our function.

Whole fuctionality is now divided into 3 different lambda function and now we are calling lambda function inside other lambda in async manner. For this we need to add following policies in existing role attached to lambda function-
AWSLambdaExecute, AWSLambdaBasicExecutionRole, AWSLambdaRole
Three functions are -
1- videoTransform: Here we do noise reduction part and save result to s3 destination bucket, and split audio and save that to s3 destination bucket and update respective URL to backend. Then call next lambada function in for watermarking
2- watermarkLogoInVideo:- Here we watermark the video with hyrr logo and then call next lambda function for creating different resolutions video
3- createVideoVersions:- This function create videos with low resolution, medium resolution as per given input video, and then update the video urls in backend.

Both watermarkLogoInVideo, createVideoVersions functions have same role as above lambda function, and layers ffmpeg and ffprobe are added for them.
