# Procedures

## Step 1: Set up Your Domain

In Amazon SES, verify the domain that you want to use to receive incoming email. For more information, see Verifying Domains in the Amazon SES Developer Guide.
Add the following MX record to the DNS configuration for your domain:

```
10 inbound-smtp.<regionInboundUrl>.amazonaws.com
```

Replace ```<regionInboundUrl>``` with the URL of the email receiving endpoint for the AWS Region that you use Amazon SES in. For a complete list of URLs, Amazon SES in the AWS General Reference.

If your account is still in the Amazon SES sandbox, submit a request to have it removed. For more information, see Moving Out of the Sandbox in the Amazon SES Developer Guide.

## Step 2: Configure Your S3 Bucket
In Amazon S3, create a new bucket. For more information, see Create a Bucket in the Amazon S3 Getting Started Guide.
Apply the following policy to the bucket:

```json
{ 
"Version": "2012-10-17", 
"Statement": [ 
    { 
    "Sid": "AllowSESPuts", 
    "Effect": "Allow", 
    "Principal": 
        { 
        "Service": "ses.amazonaws.com" 
        },
         "Action": "s3:PutObject", 
         "Resource": "arn:aws:s3:::<bucketName>/*", 
         "Condition": { 
            "StringEquals": { 
                "aws:Referer": "<awsAccountId>" 
                } 
            } 
        } 
    ] 
}
```

In the policy, make the following changes:
Replace <bucketName> with the name of your S3 bucket.
Replace <awsAccountId> with your AWS account ID.

## Step 3: Create an IAM Policy and Role
Create a new IAM Policy with the following permissions:

```json
{ "Version": "2012-10-17", "Statement": [ { "Sid": "VisualEditor0", "Effect": "Allow", "Action": [ "logs:CreateLogStream", "logs:CreateLogGroup", "logs:PutLogEvents" ], "Resource": "*" }, { "Sid": "VisualEditor1", "Effect": "Allow", "Action": [ "s3:GetObject", "ses:SendRawEmail" ], "Resource": [ "arn:aws:s3:::<bucketName>/*", "arn:aws:ses:<region>:<awsAccountId>:identity/*" ] } ] }
```

In the preceding policy, make the following changes:

Replace <bucketName> with the name of the S3 bucket that you created earlier.
Replace <region> with the name of the AWS Region that you created the bucket in.
Replace <awsAccountId> with your AWS account ID. For more information, see Create a Customer Managed Policy in the IAM User Guide.

Create a new IAM role. Attach the policy that you just created to the new role. For more information, see Creating Roles in the IAM User Guide.

## Step 4: Create the Lambda Function
In the Lambda console, create a new Python 3.7 function from scratch. For the execution role, choose the IAM role that you created earlier.

Create the following environment variables for the Lambda function:
Key	Value

`MailS3Bucket`	The name of the S3 bucket that you created earlier.

`MailS3Prefix`	The path of the folder in the S3 bucket where you will store incoming email.

`MailSender`	The email address that the forwarded message will be sent from. This address has to be verified.

`MailRecipient`	The address that you want to forward the message to.

`Region`	The name of the AWS Region that you want to use to send the email.

Under Basic settings, set the Timeout value to 30 seconds.

(Optional) 
## Step 5: Create an Amazon SNS Topic
You can optionally create an Amazon SNS topic. This step is helpful for troubleshooting purposes, or if you just want to receive additional notifications when you receive a message.

Create a new Amazon SNS topic. For more information, see Creating a Topic in the Amazon SNS Developer Guide.
Subscribe an endpoint, such as an email address, to the topic. For more information, see Subscribing an Endpoint to a Topic in the Amazon SNS Developer Guide.
## Step 6: Create a Receipt Rule Set
In the Amazon SES console, create a new Receipt Rule Set. For more information, see Creating a Receipt Rule Set in the Amazon SES Developer Guide.
In the Receipt Rule Set that you just created, add a Receipt Rule. In the Receipt Rule, add an S3 Action. Set up the S3 Action to send your email to the S3 bucket that you created earlier.
Add a Lambda action to the Receipt Rule. Configure the Receipt Rule to invoke the Lambda function that you created earlier.
For more information, see Setting Up a Receipt Rule in the Amazon SES Developer Guide.

## Step 7: Test the Function
Send an email to an address that corresponds with an address in the Receipt Rule you created earlier. Make sure that the email arrives in the correct S3 bucket. In a minute or two, the email arrives in the inbox that you specified in the `MailRecipient` variable of the Lambda function.
Troubleshooting
If you send a test message, but it is never forwarded to your destination email address, do the following:

Make sure that the Amazon SES Receipt Rule is active.
Make sure that the email address that you specified in the `MailRecipient` variable of the Lambda function is correct.
Subscribe an email address or phone number to the SNS topic. Send another test email to your domain. Make sure that SNS sends a Received notification to your subscribed email address or phone number.
Check the CloudWatch Log for your Lambda function to see if any errors occurred.
If you send a test email to your receiving domain, but you receive a bounce notification, do the following:

Make sure that the verification process for your domain completed successfully.
Make sure that the MX record for your domain specifies the correct Amazon SES receiving endpoint.
Make sure that you’re sending to an address that is handled by the receipt rule.
Costs of using this solution
The cost of implementing this solution is minimal. If you receive 10,000 emails per month, and each email is 2KB in size, you pay $1.00 for your use of Amazon SES. For more information, see Amazon SES Pricing.

You also pay a small charge to store incoming emails in Amazon S3. The charge for storing 1,000 emails that are each 2KB in size is less than one cent. Your use of Amazon S3 might qualify for the AWS Free Usage Tier. For more information, see Amazon S3 Pricing.

Finally, you pay for your use of AWS Lambda. With Lambda, you pay for the number of requests you make, for the amount of compute time that you use, and for the amount of memory that you use. If you use Lambda to forward 1,000 emails that are each 2KB in size, you pay no more than a few cents. Your use of AWS Lambda might qualify for the AWS Free Usage Tier. For more information, see AWS Lambda Pricing.

Note: These cost estimates don’t include the costs associated with purchasing a domain, since many users already have their own domains. The cost of obtaining a domain is the most expensive part of implementing this solution.