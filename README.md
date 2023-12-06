# Combination of S3, SNS, SQS and Lambda created using CloudFormation

## TL;DR
Files with the .tmp extension are placed in one folder (raw/). Process turns these files into a .log extension and then places them in a second folder (processed/). Files with other extensions are not accepted.

## How does it work
1 There is a bucket with two (2) folders: raw/ and processed/. When the file is dropped in raw/ folder, S3 event is used and notification is delivered to SNS-1.
2 SNS-1 is used as a fan for future extension. Notification about new file in S3 is delivered to SQS-1. 
3 Lambda pulls list of files to process from SQS-1. If file extension is .tmp, success status is delivered to "SNS-2". If not, failure status is delivered to SNS-2 and message is delivered to SQS-2.
4 SNS-2 sends notification to subscribers on mail
5 CloudWatch alarm is applied for SQS-2. If SQS-2 queue is grather than 5 messages, the CloudWatch alarm is run.

## Elements:
1 Bucket does not have policy. Bucket has configured event notification with prefix "raw/" and "all object creation event"
2 Event is connected to the SNS-1. 
3 SNS-1 - has access policy allowing send notifation from s3 to the sns. 
4 SQS-1 is a subcriber for notification from SNS-1
5 SQS-1 - has access policy alowing to be notified by sns.
6 Lambda has two scripts (for manual run and run by sqs trigger). Environmental variables must be configured.
Lambda role has below permissions:
-AmazonSNSFullAccess
-AmazonSQSFullAccess
-AmazonS3FullAccess
-AWSLambdaBasicExecutionRole
7 CloudWatch - alarm using "ApproximateNumberOfMessagesVisible" to minotor SQS-2
