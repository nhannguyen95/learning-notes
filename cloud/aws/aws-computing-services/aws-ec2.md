---
mindmap-plugin: basic
tags:
  - aws
  - ec2
---

# AWS EC2

## Introduction
- EC2 is Infrastructure as a Service.
- EC2 provides (access to) VMs known as instances.
- EC2 is a private service by default, it runs in the AWS Private Zone.
- An EC2 instance is configured to launch into a single [[aws-subnet]].
- EC2 is AZ resilient (instance fails if AZ fails).

## Instance Lifecycle
- **Running**
	- Instance moves to **Running** after finishing provisioning.
	- Can be **Stopped** in this state.
	- Charged on CPU/memory, disk storage, networking.
- **Stopped**
	- Can move to **Running** on restart.
	- Charged on disk storage.
- **Terminated**
	- No going back from this state.
	- No charge.

## Amazon Machine Image (AMI)
- AMI is a bundled OS for VMs that contains
	- Attached permissions which control which account can/can't use the AMI.
		- Public (AMI can be launched by all AWS accounts)
		- Explicit (specific AWS Accounts)
		- Implicit (AMI owner only)
	- Boot volume (the drive that boots the OS) of the instance
		- C-drive in Windows
		- Root volume in Linux
	- Block device mapping
		- A mapping between volumes and how the OS sees them.
	- Other pre-installed softwares.
- Can be used to create an EC2 instance,
	- The bundle can be copied to a freshly created storage volume.
	- Once the image is extracted, the volume will become a bootable drive that'll turn the VM it's attached to into a fully operational server.
- or created from an EC2 instance.
- AMIs flavors
	- RHEL
	- Ubunto
	- Windows Server
	- Amazon Linux

## Connecting to an EC2 instance
- An EC2 instance could be securely connected to with SSH.
- One of the most important SSH authentication method supported by AWS is #public-key-authentication
	- Create an #ssh-keypair which includes a public and private key.
		- The keypair could be created in AWS
		- Or you can generate your own keypair (and upload the public key to AWS)
	- The public key is copied to the SSH server (EC2 instance)
		- Once SSH server receives a public key from a user and considers it as trustworthy, the server marks the key as authorized in its *authorized_keys* files.
		- Server uses the public key to encrypt the data which can only be read by the person who holds the private key.
	- The private key remains only to the user.
		- The key is the proof of the user's identity.
		- It is called identity keys.
- Another SSH authentication method is #password-authentication.
	- Client is prompted by the server to enter a password of the corresponding account on the server to establish SSH connection.
	- This password is usually too short and easy to guess, thus not secure.
	- That's why this method is disabled by AWS by default.
		- The `/etc/ssh/sshd_config` file on Amazon Linux instance has this content: