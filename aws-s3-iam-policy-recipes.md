# AWS S3 bucket and IAM policy recipes

- [Recipes](#recipes)
	- [Anonymous GET access](#anonymous-get-access)
	- [Anonymous GET access - match HTTP referrer](#anonymous-get-access---match-http-referrer)
	- [Full access for specific IAM user/role](#full-access-for-specific-iam-userrole)
	- [GET/PUT/DELETE access to specific path within a bucket](#getputdelete-access-to-specific-path-within-a-bucket)
	- [Restricted LIST & PUT/DELETE access to specific path within a bucket](#restricted-list--putdelete-access-to-specific-path-within-a-bucket)
	- [Full access (and S3 console) for specific IAM users](#full-access-and-s3-console-for-specific-iam-users)
	- [Bucket and object delete deny](#bucket-and-object-delete-deny)
	- [Bucket enforce SSL requests only](#bucket-enforce-ssl-requests-only)
	- [CloudTrail log receive](#cloudtrail-log-receive)
	- [CloudFront origin access control (OAC) GET access](#cloudfront-origin-access-control-oac-get-access)
- [Reference](#reference)

## Recipes

### Anonymous GET access

**Type:** bucket

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "s3:GetObject",
      "Effect": "Allow",
      "Principal": {
        "AWS": "*"
      },
      "Resource": [
        "arn:aws:s3:::BUCKET_NAME/*"
      ]
    }
  ]
}
```

### Anonymous GET access - match HTTP referrer

**Type:** bucket

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "s3:GetObject",
      "Effect": "Allow",
      "Principal": {
        "AWS": "*"
      },
      "Resource": [
        "arn:aws:s3:::BUCKET_NAME/*"
      ],
      "Condition": {
        "StringLike": {
          "aws:Referer": [
            "http://domain.com/*",
            "http://www.domain.com/*"
          ]
        }
      }
    }
  ]
}
```

### Full access for specific IAM user/role

**Type:** bucket

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "s3:*",
      "Effect": "Allow",
      "Principal": {
        "AWS": [
          "arn:aws:iam::ACCOUNT_ID:user/USERNAME_A",
          "arn:aws:iam::ACCOUNT_ID:user/USERNAME_B",
          "arn:aws:iam::ACCOUNT_ID:user/USERNAME_C",
          "arn:aws:iam::ACCOUNT_ID:role/ROLE_A",
          "arn:aws:iam::ACCOUNT_ID:role/ROLE_B",
          "arn:aws:iam::ACCOUNT_ID:role/ROLE_C"
        ]
      },
      "Resource": [
        "arn:aws:s3:::BUCKET_NAME",
        "arn:aws:s3:::BUCKET_NAME/*"
      ]
    }
  ]
}
```

### GET/PUT/DELETE access to specific path within a bucket

**Type:** user/group/role

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "s3:ListBucket",
      "Effect": "Allow",
      "Resource": [
        "arn:aws:s3:::BUCKET_NAME"
      ]
    },
    {
      "Action": [
        "s3:DeleteObject",
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Effect": "Allow",
      "Resource": [
        "arn:aws:s3:::BUCKET_NAME/BUCKET_PATH/*"
      ]
    }
  ]
}
```

**Note:** The [`s3:ListBucket`](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-with-s3-actions.html#using-with-s3-actions-related-to-buckets) action against the bucket as a whole allows for the *listing* of bucket objects.

### Restricted LIST & PUT/DELETE access to specific path within a bucket

**Type:** user/group/role

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "s3:ListBucket",
      "Effect": "Allow",
      "Resource": [
        "arn:aws:s3:::BUCKET_NAME"
      ],
      "Condition": {
        "StringEquals": {
          "s3:delimiter": ["/"],
          "s3:prefix": ["","BUCKET_PATH/"]
        }
      }
    },
    {
      "Action": "s3:ListBucket",
      "Effect": "Allow",
      "Resource": [
        "arn:aws:s3:::BUCKET_NAME"
      ],
      "Condition": {
        "StringLike": {
          "s3:prefix": ["BUCKET_PATH/BUCKET_SUB_PATH/*"]
        }
      }
    },
    {
      "Action": [
        "s3:DeleteObject",
        "s3:PutObject"
      ],
      "Effect": "Allow",
      "Resource": [
        "arn:aws:s3:::BUCKET_NAME/BUCKET_PATH/BUCKET_SUB_PATH/*"
      ]
    }
  ]
}
```

**Note:** This policy effectively provides protected user folders within an S3 bucket:

- The first `s3:ListBucket` action allows *listing only* of objects at the bucket root and under `BUCKET_PATH/`.
- The second `s3:ListBucket` action allows listing of objects from the path of `BUCKET_PATH/BUCKET_SUB_PATH/` and below.
- Technique is covered [here](https://aws.amazon.com/blogs/security/writing-iam-policies-grant-access-to-user-specific-folders-in-an-amazon-s3-bucket/) under the heading _Block 2: Allow listing objects in root and home folders_.

### Full access (and S3 console) for specific IAM users

**Type:** user/group/role

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "s3:ListAllMyBuckets",
      "Effect": "Allow",
      "Resource": [
        "arn:aws:s3:::*"
      ]
    },
    {
      "Action": "s3:*",
      "Effect": "Allow",
      "Resource": [
        "arn:aws:s3:::BUCKET_NAME/*"
      ]
    }
  ]
}
```

### Bucket and object delete deny

**Type:** bucket

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "s3:DeleteBucket",
      "Effect": "Deny",
      "Principal": "*",
      "Resource": [
        "arn:aws:s3:::BUCKET_NAME"
      ]
    },
    {
      "Action": "s3:DeleteObject",
      "Effect": "Deny",
      "Principal": "*",
      "Resource": [
        "arn:aws:s3:::BUCKET_NAME/*"
      ]
    }
  ]
}
```

### Bucket enforce SSL requests only

**Type:** bucket

See: https://repost.aws/knowledge-center/s3-bucket-policy-for-config-rule

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "s3:*",
      "Effect": "Deny",
      "Principal": "*",
      "Resource": [
        "arn:aws:s3:::BUCKET_NAME/",
        "arn:aws:s3:::BUCKET_NAME/*"
      ],
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    }
  ]
}
```

### CloudTrail log receive

**Type:** bucket

See: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/create-s3-bucket-policy-for-cloudtrail.html

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "s3:GetBucketAcl",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudtrail.amazonaws.com"
      },
      "Resource": [
        "arn:aws:s3:::BUCKET_NAME/*"
      ],
      "Condition": {
        "StringEquals": {
          "aws:SourceArn": "arn:aws:cloudtrail:AWS_REGION:AWS_ACCOUNT_ID:trail/TRAIL_NAME"
        }
      }
    },
    {
      "Action": "s3:PutObject",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudtrail.amazonaws.com"
      },
      "Resource": [
        "arn:aws:s3:::BUCKET_NAME/AWSLogs/AWS_ACCOUNT_ID/*"
      ],
      "Condition": {
        "StringEquals": {
          "aws:SourceArn": "arn:aws:cloudtrail:AWS_REGION:AWS_ACCOUNT_ID:trail/TRAIL_NAME",
          "s3:x-amz-acl": "bucket-owner-full-control"
        }
      }
    }
  ]
}
```

### CloudFront origin access control (OAC) GET access

**Type:** bucket

See: https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "s3:GetObject",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Resource": [
        "arn:aws:s3:::BUCKET_NAME/*"
      ],
      "Condition": {
        "StringEquals": {
          "aws:SourceArn": "arn:aws:cloudfront::AWS_ACCOUNT_ID:distribution/DISTRIBUTION_ID"
        }
      }
    }
  ]
}
```

## Reference

- [Writing IAM Policies: How to Grant Access to an Amazon S3 Bucket](https://aws.amazon.com/blogs/security/writing-iam-policies-how-to-grant-access-to-an-amazon-s3-bucket/)
- [Writing IAM Policies: Grant Access to User-Specific Folders in an Amazon S3 Bucket](https://aws.amazon.com/blogs/security/writing-iam-policies-grant-access-to-user-specific-folders-in-an-amazon-s3-bucket/)
- Summary of `Action` types and their use:
	- https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-with-s3-actions.html
	- Grouped by bucket vs. object action types, which is very handy.
- [Amazon S3 condition key examples](https://docs.aws.amazon.com/AmazonS3/latest/userguide/amazon-s3-policy-keys.html)
- [Example IAM identity-based policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_examples.html)
