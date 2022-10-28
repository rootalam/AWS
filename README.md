# Copy AWS S3 bucket across account

We will copy AWS S3 bucket from one AWS account to another account AWS account.
1. In source account, create an IAM custom policy that grants an IAM user or role proper permissions. The IAM user must have access to retrieve objects from the source bucket and put objects back into the destination bucket. You can use an IAM policy similar to the following:

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetObject"
      ],
      "Resource": [
        "arn:aws:s3:::SOURCE-BUCKET-NAME",
        "arn:aws:s3:::SOURCE-BUCKET-NAME/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:PutObject",
        "s3:PutObjectAcl"
      ],
      "Resource": [
        "arn:aws:s3:::DESTINATION-BUCKET-NAME",
        "arn:aws:s3:::DESTINATION-BUCKET-NAME/*"
      ]
    }
  ]
}
```
Note: Please replace SOURCE-BUCKET-NAME and DESTINATION-BUCKET-NAME as per your scenario.

2. In source account, create an IAM user and attach newly created policy in step 1. Download access key of this user and configure it on aws cli.
3. If you want to perform this activity from EC2 instance of source account then create one role and assign newly created policy to that role. Then           attach that role to EC2 instance. (For this step 2 is not required)
4. In the destination account, set S3 Object Ownership on the destination bucket to bucket owner preferred. After you set S3 Object Ownership, new objects uploaded with the access control list (ACL) set to bucket-owner-full-control are automatically owned by the bucket's account.
5. In the destination account, add bucket policy to the destination bucket to grant permissions to upload objects from source account. You can use bucket policy similar to the following:
```
{
  "Version": "2012-10-17",
  "Id": "Policy1611277539797",
  "Statement": [
    {
      "Sid": "Stmt1611277535086",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::1111111111:user/UserName"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::DESTINATION-BUCKET-NAME/*",
      "Condition": {
        "StringEquals": {
          "s3:x-amz-acl": "bucket-owner-full-control"
        }
      }
    },
    {
      "Sid": "Stmt1611277877767",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::1111111111:user/UserName"
      },
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::DESTINATION-BUCKET-NAME"
    }
  ]
}
```
Note: Replace DESTINATION-BUCKET-NAME with your destination bucket name and arn:aws:iam::1111111111:user/UserName with newly created user's ARN.
      If you are using IAM Roles then replace arn:aws:iam::1111111111:user/UserName with Role ARN.

6. After that use newly created user or EC2 instance in which you have attach newly created IAM role to copy bucket from source to destination account. Command is as given below:
 ```
aws s3 sync s3://SOURCE-BUCKET-NAME/ s3://DESTINATION-BUCKET-NAME/ --acl bucket-owner-full-control
```
