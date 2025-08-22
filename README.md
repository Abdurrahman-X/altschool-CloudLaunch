# altschool-CloudLaunch

# CloudLaunch - AWS Fundamentals Project

A comprehensive AWS cloud platform demonstrating static website hosting, IAM access control, and VPC network design using AWS core services.

## 🚀 Live Demo

**Website**: `http://abdurrahman-cloudlaunch-site-bucket.s3-website-us-east-1.amazonaws.com/`

**CloudFront URL**: `https://d2o550r9vect4h.cloudfront.net`

## 📋 Project Overview

CloudLaunch is a lightweight platform that showcases AWS fundamentals through two main components:

1. **Task 1**: Static Website Hosting with S3 and IAM access control
2. **Task 2**: VPC Network Design with multi-tier architecture

This project demonstrates an understanding of AWS core services, including S3, IAM, VPC, Route Tables, Security Groups, and Internet Gateways, while implementing security best practices.


## 📋 Task 1: S3 Static Website Hosting + IAM Implementation

### What I Did - Step by Step

#### Step 1: Created Three S3 Buckets

**1.1 abdurrahman-cloudlaunch-site-bucket (Public Website)**
- Created bucket in the us-east-1 region
- **Disabled "Block all public access"** to allow public website hosting
- Enabled static website hosting in Properties
- Set index document to `index.html`
- Uploaded basic HTML website with CloudLaunch branding
- Applied bucket policy to allow public read access

**1.2 abdurrahman-cloudlaunch-private-bucket (Private Storage)**
- Created bucket with default private settings
- **Kept "Block all public access" enabled** for security
- No public access - only accessible via IAM permissions
- Intended for storing internal private documents

**1.3 abdurrahman-cloudlaunch-visible-only-bucket (List-Only Access)**
- Created bucket with default private settings
- Configured to demonstrate limited IAM permissions
- IAM user can see the bucket exists, but cannot access the contents

#### Step 2: Configured S3 Bucket Policy for Public Website

Applied the following policy to `abdurrahman-cloudlaunch-site-bucket`:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadForCloudLaunch",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::abdurrahman-cloudlaunch-site-bucket/*"
        }
    ]
}
```

**Why this policy**: Allows anonymous users to read website files while maintaining security on other buckets.

#### Step 3: Created IAM User with Custom Permissions

**3.1 Created User**
- Username: `abdurrahman-cloudlaunch-user`
- Enabled console access with a custom password
- **Enforced password change on first login** for security

**3.2 Created Custom IAM Policy**
- Policy name: `CloudLaunchUserPolicy`
- Implemented the principle of least privilege
- **No delete permissions anywhere** for data protection
- Limited access to only the three CloudLaunch buckets
- Added VPC read-only permissions for Task 2

#### Step 4: IAM Policy Implementation Details

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ListAllBuckets",
            "Effect": "Allow",
            "Action": "s3:ListAllMyBuckets",
            "Resource": "*",
            "Comment": "Allows user to see all buckets in S3 console"
        },
        {
            "Sid": "ListSpecificBuckets",
            "Effect": "Allow",
            "Action": "s3:ListBucket",
            "Resource": [
                "arn:aws:s3:::abdurrahman-cloudlaunch-site-bucket",
                "arn:aws:s3:::abdurrahman-cloudlaunch-private-bucket",
                "arn:aws:s3:::abdurrahman-cloudlaunch-visible-only-bucket"
            ],
            "Comment": "Allows user to list contents of CloudLaunch buckets"
        },
        {
            "Sid": "GetObjectSiteBucket",
            "Effect": "Allow",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::abdurrahman-cloudlaunch-site-bucket/*",
            "Comment": "Read-only access to website files"
        },
        {
            "Sid": "GetPutObjectPrivateBucket",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::abdurrahman-cloudlaunch-private-bucket/*",
            "Comment": "Read and write access to private documents, but no delete"
        },
        {
            "Sid": "VPCReadOnlyAccess",
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeVpcs",
                "ec2:DescribeSubnets",
                "ec2:DescribeRouteTables",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeInternetGateways",
                "ec2:DescribeNetworkAcls",
                "ec2:DescribeVpcEndpoints",
                "ec2:DescribeAvailabilityZones"
            ],
            "Resource": "*",
            "Comment": "Read-only access to VPC components for Task 2"
        }
    ]
}
```

### Bonus: CloudFront Distribution

**Why I implemented CloudFront**:
- Provides HTTPS encryption for the static website
- Global content delivery for faster loading times
- Professional deployment practice

**Configuration**:
- Origin: S3 static website endpoint
- SSL certificate: CloudFront default certificate
- Caching: Default CloudFront caching policies

---

## 📋 Task 2: VPC Network Design Implementation

### What I Did - Step by Step

#### Step 1: Created VPC Foundation

**1.1 Created VPC**
- Name: `abdurrahman-cloudlaunch-vpc`
- CIDR Block: `10.0.0.0/16`
- Provides 65,536 IP addresses for scalability
- **Why this CIDR**: Standard private network range, large enough for future expansion

#### Step 2: Designed Three-Tier Subnet Architecture

**2.1 Public Subnet**
- Name: `abdurrahman-cloudlaunch-public-subnet`
- CIDR: `10.0.1.0/24` (254 usable IPs)
- **Purpose**: Future load balancers, bastion hosts, NAT gateways
- **Internet Access**: Yes (via Internet Gateway)

**2.2 Application Subnet**
- Name: `abdurrahman-cloudlaunch-app-subnet`
- CIDR: `10.0.2.0/24` (254 usable IPs)
- **Purpose**: Application servers, web servers, API services
- **Internet Access**: No (private subnet)

**2.3 Database Subnet**
- Name: `abdurrahman-cloudlaunch-db-subnet`
- CIDR: `10.0.3.0/28` (14 usable IPs)
- **Purpose**: Database servers, data storage services
- **Internet Access**: No (private subnet)

#### Step 3: Implemented Internet Connectivity

**3.1 Created Internet Gateway**
- Name: `abdurrahman-cloudlaunch-igw`
- **Attached to**: `abdurrahman-cloudlaunch-vpc`
- **Purpose**: Provides internet access for public subnet

#### Step 4: Configured Network Routing

**4.1 Public Route Table**
- Name: `abdurrahman-cloudlaunch-public-rt`
- **Routes Configured**:
  - `10.0.0.0/16` → Local (VPC internal traffic)
  - `0.0.0.0/0` → `abdurrahman-cloudlaunch-igw` (Internet traffic)
- **Associated with**: `abdurrahman-cloudlaunch-public-subnet`

**4.2 Application Route Table**
- Name: `abdurrahman-cloudlaunch-app-rt`
- **Routes Configured**:
  - `10.0.0.0/16` → Local (VPC internal traffic only)
- **No internet route** - ensures privacy
- **Associated with**: `abdurrahman-cloudlaunch-app-subnet`

**4.3 Database Route Table**
- Name: `abdurrahman-cloudlaunch-db-rt`
- **Routes Configured**:
  - `10.0.0.0/16` → Local (VPC internal traffic only)
- **No internet route** - maximum security
- **Associated with**: `abdurrahman-cloudlaunch-db-subnet`

#### Step 5: Implemented Security Groups

**5.1 Application Security Group**
- Name: `abdurrahman-cloudlaunch-app-sg`
- **Inbound Rules**:
  - HTTP (80) from VPC CIDR (10.0.0.0/16)
- **Outbound Rules**: All traffic (default)

**5.2 Database Security Group**
- Name: `abdurrahman-cloudlaunch-db-sg`
- **Inbound Rules**:
  - MySQL/Aurora (3306) from App Subnet (10.0.2.0/24)
  - **Why**: Only application servers can access the database
- **Outbound Rules**: All traffic (default)

**Project Author**: Abdurrahman Abolaji
**StudentID**: ALT/SOE/024/2394
**Submission Date**: 22nd August 2025  
**AWS Region**: us-east-1  
**Assignment**: AltSchool Cloud Engineering Third Semester Assignment 1
