# Lab: Write the Node IAM Role

## Goal

Create the IAM role your worker nodes assume to join the cluster and do their work. By the end you have an IAM role trusted by the EC2 service and granted the three AWS-managed policies every EKS node needs. This role is a prerequisite for adding nodes: without it, an instance has no identity to register with.

## Starting point

You continue from the applied project: the inputs, locals, provider, VPC, the cluster role, and the control plane are already in place and applied. You add one new file for the node role alongside them.

## Configuration to target

- **Role name:** the cluster name followed by `-node` (so `eks-foundations-node`), built from the existing name local.
- **Who may assume it (trust):** the EC2 service, identified by the service principal `ec2.amazonaws.com`, allowed to perform `sts:AssumeRole`. A node is an EC2 instance, so EC2 is the trusted party.
- **What it may do (permissions):** attach three AWS-managed policies, one per job:
  - `AmazonEKSWorkerNodePolicy`: join and talk to the cluster.
  - `AmazonEKS_CNI_Policy`: run the pod-networking agent.
  - `AmazonEC2ContainerRegistryPullOnly`: pull container images.
- **Tags:** the project base tags, consistent with the rest of the project.

## Tasks

1. **Write the trust policy.** Define an assume-role policy document that allows the EC2 service principal to assume the role. This is the "who may assume it" half.
2. **Create the role.** Create an IAM role named from the cluster name with the `-node` suffix, using that trust policy, and tag it.
3. **Attach the three managed policies.** Attach all three AWS-managed policies to the role. A single attachment iterating over the set of policy ARNs is cleaner than three separate blocks.
4. **Apply.** Review the plan (an IAM role plus three policy attachments) and apply.

## Done when

- `terraform apply` completes, creating the IAM role and the three policy attachments.
- `terraform validate` reports the configuration is valid.
- Looking the role up by name shows it exists, trusts the EC2 service, and has three managed policies attached.

## Note

IAM roles and policy attachments are free, so this lab adds nothing to your bill. No nodes exist yet: this is the identity your nodes will assume when you add worker capacity next.
