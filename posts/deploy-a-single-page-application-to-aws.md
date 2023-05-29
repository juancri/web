---
title: Deploy a single page application to AWS
date: 2023-05-29
tags:
  - english
  - software
  - cloud
  - aws
  - development
  - frontend
layout: layouts/post.njk
---

## Introduction

The goal

Challenges

- Two types of routing
- All routes are served same HTML file

## Architecture

- CloudFront
- Route53
- Amazon Certificate Manager
- S3

## Steps

- Create a simple Vue 3 app
- Read input (read -p)
- Run the rest...

```
# Create bucket
AWS_PROFILE=treesoft aws s3api create-bucket \
  --bucket treesoft-info-web-dev \
  --region us-east-1

# Enable public reads
AWS_PROFILE=treesoft aws s3api put-public-access-block \
  --bucket treesoft-info-web-dev \
  --public-access-block-configuration "BlockPublicPolicy=false"
AWS_PROFILE=treesoft aws s3api put-bucket-policy \
  --bucket treesoft-info-web-dev \
  --policy '{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::treesoft-info-web-dev/*"
            ]
        }
    ]
}'

# Enable website
AWS_PROFILE=treesoft aws s3 website s3://treesoft-info-web-dev/ \
  --index-document index.html

# From the info-web directory
# Build
npm run build

# Sync
AWS_PROFILE=treesoft aws s3 sync \
  dist/ \
  s3://treesoft-info-web-dev/

# Here, we create the certificate manually
# using AWS Certificate Manager

# Create CloudFront distribution config file
tee -a dist-config.json > /dev/null <<EOT
{
    "CallerReference": "cli",
    "Aliases": {
        "Quantity": 1,
        "Items": [
            "info.dev.treesoft.cl"
        ]
    },
    "DefaultRootObject": "",
    "Origins": {
        "Quantity": 1,
        "Items": [
            {
                "Id": "s3-treesoft-info-web-dev",
                "DomainName": "treesoft-info-web-dev.s3-website-us-east-1.amazonaws.com",
                "OriginPath": "",
                "CustomHeaders": {
                    "Quantity": 0
                },
                "CustomOriginConfig": {
                    "HTTPPort": 80,
                    "HTTPSPort": 443,
                    "OriginProtocolPolicy": "http-only",
                    "OriginSslProtocols": {
                        "Quantity": 3,
                        "Items": [
                            "TLSv1",
                            "TLSv1.1",
                            "TLSv1.2"
                        ]
                    },
                    "OriginReadTimeout": 30,
                    "OriginKeepaliveTimeout": 5
                },
                "ConnectionAttempts": 3,
                "ConnectionTimeout": 10,
                "OriginShield": {
                    "Enabled": false
                },
                "OriginAccessControlId": ""
            }
        ]
    },
    "OriginGroups": {
        "Quantity": 0
    },
    "DefaultCacheBehavior": {
        "TargetOriginId": "s3-treesoft-info-web-dev",
        "TrustedSigners": {
            "Enabled": false,
            "Quantity": 0
        },
        "TrustedKeyGroups": {
            "Enabled": false,
            "Quantity": 0
        },
        "ViewerProtocolPolicy": "redirect-to-https",
        "AllowedMethods": {
            "Quantity": 2,
            "Items": [
                "HEAD",
                "GET"
            ],
            "CachedMethods": {
                "Quantity": 2,
                "Items": [
                    "HEAD",
                    "GET"
                ]
            }
        },
        "SmoothStreaming": false,
        "Compress": false,
        "LambdaFunctionAssociations": {
            "Quantity": 0
        },
        "FunctionAssociations": {
            "Quantity": 0
        },
        "FieldLevelEncryptionId": "",
        "ForwardedValues": {
            "QueryString": false,
            "Cookies": {
                "Forward": "none"
            },
            "Headers": {
                "Quantity": 0
            },
            "QueryStringCacheKeys": {
                "Quantity": 0
            }
        },
        "MinTTL": 0,
        "DefaultTTL": 86400,
        "MaxTTL": 31536000
    },
    "CacheBehaviors": {
        "Quantity": 0
    },
    "CustomErrorResponses": {
        "Quantity": 1,
        "Items": [
            {
                "ErrorCode": 404,
                "ResponsePagePath": "/index.html",
                "ResponseCode": "200",
                "ErrorCachingMinTTL": 3600
            }
        ]
    },
    "Comment": "",
    "Logging": {
        "Enabled": false,
        "IncludeCookies": false,
        "Bucket": "",
        "Prefix": ""
    },
    "PriceClass": "PriceClass_All",
    "Enabled": true,
    "ViewerCertificate": {
        "CloudFrontDefaultCertificate": false,
        "ACMCertificateArn": "arn:aws:acm:us-east-1:230767153216:certificate/f06ae507-4439-45e3-85be-29d7e4c34c8e",
        "SSLSupportMethod": "sni-only",
        "MinimumProtocolVersion": "TLSv1",
        "Certificate": "arn:aws:acm:us-east-1:230767153216:certificate/f06ae507-4439-45e3-85be-29d7e4c34c8e",
        "CertificateSource": "acm"
    },
    "Restrictions": {
        "GeoRestriction": {
            "RestrictionType": "none",
            "Quantity": 0
        }
    },
    "WebACLId": "",
    "HttpVersion": "http2",
    "IsIPV6Enabled": true,
    "ContinuousDeploymentPolicyId": "",
    "Staging": false
}
EOT

# Create CloudFront distribution
AWS_PROFILE=treesoft aws cloudfront create-distribution --distribution-config file://dist-config.json

# Here we manually create a CNAME from the hostname to the CloudFront distribution URL
```
