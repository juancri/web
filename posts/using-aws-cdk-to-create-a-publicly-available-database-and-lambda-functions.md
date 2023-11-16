---
title: Using AWS CDK to create publicly available database and lambda functions
date: 2023-11-16
tags:
  - english
  - aws
  - lambda
  - cdk
  - cloud
  - software
  - development
  - postgresql
layout: layouts/post.njk
---

## Introduction

The goal is to create an RDS PostgreSQL database and lambda functions that meet the following requirements:

- The database is available publicly for specific IP addresses
- The database is available for the lambda functions
- The lambda functions can connect to PostgreSQL
- The lambda functions can connect to CloudWatch, SES and other AWS services

The reasons why this is not that easy:

- RDS databases run inside a VPC
- Lambda functions can run inside a VPC but only on private subnets
- Private subnets need a route table to connect to other private resources and to the Internet by using a NAT gateway
- Public subnets need a route table to connect to other private resources and to the Internet by using an Internet gateway
- Lambda functions and the database each need a security group

Give these requirements and constraints, this is the diagram of what we need to build:

```

                 Internet
                /        \
     NAT gateway          Internet gateway
          |                        |
|---VPC---|------------------------|----------------|
|         |                        |                |
|  |-Private-subnet---|    |-Public-subnet-------|  |
|  |                  |    |                     |  |
|  | Lambda functions |----| PostgreSQL database |  |
|  |   (lambda sg)    |    |   (postgresql sg)   |  |
|  |------------------|    |---------------------|  |
|                                                   |
|---------------------------------------------------|

```

## CDK

I initially built this architecture manually for the dev environment of a project. Now I need to keep track of all the changes and document the architecture when mounting the production environment so I'm using CDK with TypeScript.

