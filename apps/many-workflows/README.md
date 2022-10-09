# Delivery instructions for many-workflows application

> check the [main repository](https://github.com/nikita-akuity/demo-workflow/tree/main/manifests/many-workflows) for the application description

## Requirements
Some resources should be created prior to the application deployment:

* AWS SQS queue
* AWS S3 bucket
  * [configured](https://docs.aws.amazon.com/AmazonS3/latest/userguide/NotificationHowTo.html) S3 notifications to the SQS queue
* MySQL database
* AWS Secrets Manager secrets with mysql credentials
* EKS cluster with [configured](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) IRSA
* AWS IAM policy (or policies) with access to all mentioned resources  
  e.g. of the policy that includes everything:
  ```
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Sid": "VisualEditor0",
        "Effect": "Allow",
        "Action": [
          "sqs:DeleteMessage",
          "sqs:GetQueueUrl",
          "sqs:ReceiveMessage",
          "sqs:GetQueueAttributes",
          "sqs:ListQueueTags",
          "sqs:ListDeadLetterSourceQueues",
          "s3:ListBucket",
          "s3:PutObject",
          "s3:GetObject",
          "s3:DeleteObject"
          "secretsmanager:DescribeSecret",
          "secretsmanager:ListSecretVersionIds",
          "secretsmanager:GetResourcePolicy",
          "secretsmanager:GetSecretValue",
        ],
        "Resource": [
          "arn:aws:s3:::<bucket_name>/*",
          "arn:aws:s3:::<bucket_name>",
          "arn:aws:secretsmanager:<region>:<aws_id>:secret:<secrets_prefix>/*",
          "arn:aws:sqs:<region>:<aws_id>:<sqs_queue_name>"
        ]
      },
      {
        "Sid": "VisualEditor1",
        "Effect": "Allow",
        "Action": "sqs:ListQueues",
        "Resource": "*"
      }
    ]
  }
  ```

## Environment specific resources

* [Eventbus](https://argoproj.github.io/argo-events/concepts/eventbus/) for Argo Events to function properly
* Serice accounts and roles:
  * for running workflows
    *  with AWS S3 access
    *  with `executor` role 
  * for running sensors
    * with AWS SQS access
    * with `operate-workflow` role
  * for accessing secrets
    * with AWS Secrets Manager access
  
  For simpicity all these serviceaccounts and roles were squashed into one or two. Don't do that in real applications, pay some attention to security.
* Configmaps and secrets
