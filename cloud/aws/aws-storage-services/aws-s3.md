---
mindmap-plugin: basic
tags:
  - aws
  - "#s3"
---

# AWS S3

## Introduction
- S3 is a global service (the service runs from AWS public zone).
- S3 is regional resilient (in terms of the data you store).
- S3 lets you store and retrieve data organized as **objects** via API reachable over HTTPS.
	- Filename of an object is its **key** and can be 1024-byte long.
	- An object can be from *0 to 5TB* in size.
- S3 stores objects in a container called a **bucket**.
	- A bucket exists inside 1 region.
		- Data in a bucket never leave the region unless configured otherwise.
	- A bucket name
		- must be globally unique.
		- from 3-63 character long.
	- You can have up to 100 buckets as soft limit and 1000 as hard limit (support request) per AWS account.
	- A bucket can store unlimited number of objects.
	- Buckets have no structure, they are a flat structure (all objects are under the same level).
		- "Folders" in S3 can be represented by pathing object key (e.g. `/path/to/object/key.blob`)
		- UI still represents `path`, `to`, `object` as folders.
	- Buckets are private by default.