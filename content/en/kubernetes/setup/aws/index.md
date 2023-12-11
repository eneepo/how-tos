---
title: "Kubernetes with AWS EKS"
linkTitle: "AWS EKS"
description: "How to create a Kubernetes cluster using AWS EKS"
date: 2023-09-23T08:28:06Z
categories: []
tags: [aws, eks]
---

## EKS
Amazon Elastic Kubernetes Service (Amazon EKS) is a managed Kubernetes service that simplifies the deployment, management, and scaling of Kubernetes clusters on AWS infrastructure.

You can create a Kubernetes cluster on AWS Elastic Kubernetes Service using either the AWS Management Console (UI) or the `eksctl` CLI. These are two distinct methods to provision and manage your EKS clusters:

### AWS Management Console (UI)
Using the AWS Management Console, you can create an EKS cluster through a web-based interface. This method is more user-friendly and may be preferred by those who are less comfortable with the command line.

To create an EKS cluster via the AWS Management Console:
    - Log in to the AWS Management Console.
    - Navigate to the EKS service.
    - Click "Create Cluster."
    - Follow the on-screen prompts to configure your cluster, including VPC settings, worker node groups, and security options.

### eksctl CLI
`eksctl` is a command-line tool designed specifically for creating and managing EKS clusters. It provides more fine-grained control and flexibility compared to the AWS Management Console.


#### eksctl Installation
To create an EKS cluster using `eksctl`, you'll need to install `eksctl` on your local machine and configure it:

```bash
# for ARM systems, set ARCH to: `arm64`, `armv6` or `armv7`
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH

curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"

# (Optional) Verify checksum
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check

tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz

sudo mv /tmp/eksctl /usr/local/bin
```

For other ways to install `eksctl` visit the [eksctl documentation](https://eksctl.io/introduction/#installation)

#### Create EKS cluster
Here's a simplified example of creating an EKS cluster using `eksctl`:

```bash
eksctl create cluster --name dev-cluster --version 1.27 --region us-east-1 --nodegroup-name standard-workers --node-type t3.micro --nodes 3 --nodes-min 1 --nodes-max 4 --managed
```

Here's a breakdown of each flag used in the `eksctl create cluster` command above:

- `--name dev-cluster`: This flag specifies the name of the EKS cluster you want to create, which in this case is "dev-cluster."
- `--version 1.27`: This flag specifies the Kubernetes version you want to use for your EKS cluster. In this example, the version is set to 1.27.
- `--region us-east-1`: This flag indicates the AWS region where you want to create your EKS cluster. In this case, the region is set to "us-east-1."
- `--nodegroup-name standard-workers`: This flag sets the name of the node group that will be created as part of your EKS cluster. Node groups are groups of worker nodes that can be managed collectively.
- `--node-type t3.micro`: This flag specifies the EC2 instance type for the worker nodes in your node group. In this example, the worker nodes will use the "t3.micro" instance type, which is a cost-effective and small instance type.
- `--nodes 3`: This flag sets the initial number of worker nodes to create in the node group. In this case, three worker nodes will be launched initially.
- `--nodes-min 1`: This flag specifies the minimum number of worker nodes that should be maintained in the node group. Even if your cluster is scaled down due to low demand, at least one worker node will always be running.
- `--nodes-max 4`: This flag sets the maximum number of worker nodes that can be scaled up to in the node group. If your cluster experiences high demand, it can scale up to a maximum of four worker nodes.
- `--managed`: This flag indicates that you want to use managed node groups for your worker nodes. Managed node groups are easier to manage because AWS takes care of aspects like scaling and node replacement.