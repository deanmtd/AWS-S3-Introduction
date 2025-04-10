# AWS-S3-Introduction
# Amazon S3 Project: Understanding S3 Core Concepts

## Introduction

This project demonstrates my comprehensive understanding of Amazon S3 (Simple Storage Service), one of AWS's foundational services. Through hands-on examples, I explore the essential S3 features relevant for AWS certification and real-world implementation.

## Table of Contents

1. [Creating S3 Buckets](#creating-s3-buckets)
2. [Uploading Objects to S3](#uploading-objects-to-s3)
3. [Working with Bucket Policies](#working-with-bucket-policies)
4. [Static Website Hosting](#static-website-hosting)
5. [S3 Versioning](#s3-versioning)
6. [S3 Replication](#s3-replication)
7. [S3 Storage Classes](#s3-storage-classes)
8. [Summary and Best Practices](#summary-and-best-practices)

## Creating S3 Buckets

Creating an S3 bucket requires understanding several key concepts:

### Region Selection
When creating a bucket, I first selected a region (eu-north-1 in this example). While S3 provides a global view of all buckets across regions, each bucket exists in one specific region.

### Bucket Types
AWS offers two types of buckets:
- **General Purpose buckets** (recommended for most use cases)
- **Directory buckets** (for specific use cases, not covered in AWS certification exams)

If the bucket type selection isn't available in your region, the bucket will automatically be created as a general purpose bucket.

### Bucket Naming
The most critical rule for bucket naming is **global uniqueness**. Bucket names must be unique across:
- All AWS regions
- All AWS accounts
- All time (even deleted bucket names remain reserved)

For this reason, I used a naming pattern that included my name, the project, and a version number: `stephane-demo-s3-v5`.

### Key Security Settings

When creating the bucket, I maintained the default security settings:
- **Object Ownership**: ACL disabled (recommended)
- **Block Public Access**: All public access blocked (highest security)
- **Bucket Versioning**: Initially disabled (will enable later)
- **Default Encryption**: Server-side encryption with Amazon S3 managed keys (SSE-S3)
- **Bucket Key**: Enabled

## Uploading Objects to S3

After bucket creation, I uploaded objects to demonstrate S3's storage capabilities:

1. I uploaded a file called `coffee.jpg` (100KB) to the bucket
2. Navigated to the object details page to examine its properties
3. Demonstrated two methods of accessing the file:
   - Using the S3 console's "Open" feature (works because I own the bucket)
   - Trying the public URL (initially fails with "Access Denied" due to default security settings)

### Working with Folders
S3 doesn't actually have folders in the traditional sense - it simulates a folder structure through prefixes in object keys. However, the S3 console provides a folder-like interface for organization:

1. I created a folder called "images"
2. Uploaded `beach.jpg` into the images folder
3. Demonstrated how to navigate the folder structure
4. Showed how to delete folders, which actually deletes all objects with that prefix

## Working with Bucket Policies

To make objects publicly accessible, I implemented a bucket policy:

### Allowing Public Access
1. First, I modified the bucket settings to allow public access by editing the "Block Public Access" settings
2. I acknowledged the security warning, understanding that this is appropriate only for content that should be publicly available

### Creating a Bucket Policy
1. Navigated to the Permissions tab and selected Bucket Policy
2. Used the AWS Policy Generator to create an appropriate policy:
   - Effect: Allow
   - Principal: * (everyone)
   - Action: s3:GetObject
   - Resource: ARN of my bucket followed by "/*" to indicate all objects

The resulting policy:

```json
{
  "Id": "Policy1649248504940",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1649248501271",
      "Action": [
        "s3:GetObject"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::stephane-demo-s3-v5/*",
      "Principal": "*"
    }
  ]
}
```

After applying this policy, I confirmed that my `coffee.jpg` image was now publicly accessible via its object URL.

## Static Website Hosting

S3 offers the ability to host static websites directly from a bucket, which I demonstrated by:

1. Uploading two image files: `coffee.jpg` and `beach.jpg`
2. Enabling static website hosting under the bucket Properties tab
3. Setting `index.html` as the index document
4. Creating and uploading an `index.html` file with basic HTML content referencing the images
5. Accessing the website using the bucket website endpoint URL
6. Verifying that all linked resources (images) were properly loading

My simple `index.html` file contained:
```html
<html>
<body>
<h1>I love coffee</h1>
<p>Hello world!</p>
<img src="coffee.jpg">
</body>
</html>
```

This demonstrates how S3 can serve as a cost-effective hosting solution for static websites without requiring web servers.

## S3 Versioning

Versioning is a critical S3 feature that maintains multiple versions of objects to protect against accidental deletions and modifications:

1. I enabled versioning under the bucket Properties tab
2. Modified my `index.html` file to say "I REALLY love coffee" instead of "I love coffee"
3. Uploaded the updated file to S3, overwriting the previous version
4. Verified the website displayed the updated content
5. Used the "Show versions" toggle to reveal both versions of the file:
   - The original version (with null version ID, uploaded before versioning was enabled)
   - The new version (with a unique version ID)

### Working with Versions and Delete Markers

I demonstrated version management through:

1. **Rollback**: Deleting the newer version (permanent deletion of a specific version) to restore the previous content
2. **Delete markers**: Deleting an object in a versioned bucket creates a "delete marker" rather than permanently removing content
3. **Restoration**: Removing the delete marker to restore the previously "deleted" object

This exercise illustrated how versioning provides robust protection against data loss while maintaining a history of changes.

## S3 Replication

S3 replication allows automatic copying of objects between buckets, which I demonstrated by:

1. Creating two buckets:
   - Origin bucket: `s3-stephane-bucket-origin-v2` in eu-west-1
   - Replica bucket: `s3-stephane-bucket-replica-v2` in us-east-1
2. Enabling versioning on both buckets (required for replication)
3. Uploading an initial test file (`beach.jpg`) to the origin bucket
4. Configuring replication:
   - Created a replication rule named "DemoReplicationRule"
   - Set the source as all objects in the origin bucket
   - Selected the replica bucket as the destination
   - Created a new IAM role for replication permissions
   - Opted not to replicate existing objects
5. Testing replication by:
   - Uploading a new file (`coffee.jpg`) to the origin bucket
   - Verifying it appeared in the replica bucket with identical version IDs
   - Uploading a new version of `beach.jpg` and confirming it replicated
   - Enabling delete marker replication in the replication settings
   - Verifying that delete markers get replicated while permanent deletions do not

This exercise demonstrated Cross-Region Replication (CRR), which provides geographical redundancy, compliance support, and improved access latency for global users.

## S3 Storage Classes

S3 offers multiple storage classes to optimize cost and performance. I explored these by:

1. Creating a new bucket named `s3-storage-classes-demos-2022`
2. Uploading an object (`coffee.jpg`) while examining available storage classes:
   - **S3 Standard**: Default class with high durability, availability, and performance
   - **S3 Intelligent-Tiering**: Automatically moves objects between tiers based on access patterns
   - **S3 Standard-IA**: For infrequently accessed data with low latency requirements
   - **S3 One-Zone-IA**: Lower cost option storing data in a single AZ
   - **Glacier Instant Retrieval**: For archive data needing immediate access
   - **Glacier Flexible Retrieval**: For archive data with retrieval times from minutes to hours
   - **Glacier Deep Archive**: Lowest-cost storage for rarely accessed data with retrieval times of hours
   - **Reduced Redundancy**: Deprecated storage class

3. Demonstrating storage class transitions:
   - Uploading an object with Standard-IA class
   - Changing the storage class to One-Zone-IA through the object properties
   - Further changing it to Glacier Instant Retrieval

4. Creating an automated Lifecycle Rule ("DemoRule") to automatically transition objects between storage classes:
   - Move to Standard-IA after 30 days
   - Move to Intelligent-Tiering after 60 days
   - Move to Glacier Flexible Retrieval after 180 days

This exercise demonstrated how storage classes can be leveraged to optimize costs based on data access patterns and retention requirements.

## Summary and Best Practices

Throughout this project, I've demonstrated proficiency with Amazon S3's core features that are critical for AWS certification and real-world application:

### Key Concepts Mastered:
- Bucket creation and naming conventions
- Object upload and management
- Security configuration through bucket policies
- Static website hosting
- Data protection through versioning
- Geographical redundancy with replication
- Cost optimization with storage classes and lifecycle policies

### Best Practices:
1. **Security**: Block public access by default, use bucket policies judiciously
2. **Data Protection**: Enable versioning for important buckets
3. **Redundancy**: Use replication for critical data
4. **Cost Optimization**: Implement lifecycle policies to transition data to appropriate storage classes
5. **Organization**: Use prefixes (folders) to structure data logically

This project demonstrates my comprehensive understanding of Amazon S3 and readiness to implement S3-based solutions in production environments and pass relevant AWS certification exams.
