# Lab: Write the Root Inputs and the VPC

## Goal

Give the project its shared inputs and build the network the cluster will live in. By the end you have the root variables, the shared locals, a wired-up provider, and a VPC suitable for EKS: public and private subnets across two availability zones, a NAT gateway for the private subnets, and the subnet tags EKS needs to place load balancers. This is the first lab that creates real, billable infrastructure.

## Starting point

You begin from the initialized `infra/` project: `versions.tf`, `providers.tf`, and `backend.tf` are in place, and Terraform is already initialized against your S3 backend. `providers.tf` currently declares the AWS provider with no arguments. You will add the remaining files alongside these and extend the provider.

## Configuration to target

- **Region:** read from the devcontainer's `AWS_REGION` rather than a Terraform variable. The provider picks it up from the environment.
- **Cluster name:** `eks-foundations`, as the default of a `cluster_name` variable. It also serves as the network's name and a resource prefix.
- **VPC CIDR:** `10.0.0.0/16`, as the default of a `vpc_cidr` variable.
- **Availability zones:** the first two available zones in the region that require no opt-in. You can leverage data sources instead of hard-coding them.
- **Subnets:** one public and one private subnet per availability zone, derived from the VPC CIDR. Nodes and control-plane interfaces live in the private subnets; the public subnets exist for load balancers.
- **NAT gateway:** a single shared NAT gateway for the whole VPC. This is the deliberate cost choice for a lab; production would run one per zone.
- **Subnet tags for EKS load balancer placement:**
  - Public subnets: `kubernetes.io/role/elb` = `1` (internet-facing load balancers).
  - Private subnets: `kubernetes.io/role/internal-elb` = `1` (internal load balancers).
- **Base tags:** `Project` = `eks-foundations` and `ManagedBy` = `terraform`, applied to every resource, and mergeable with an optional additional-tags map so callers can add their own.
- **Networking approach:** you can use the community VPC module if you do not wish to hand-write subnets, route tables, or gateways. That being said, coding everything manually would be a great exercise too! 🙂

## Tasks

1. **Define the root inputs.** Create the variables for cluster name, VPC CIDR, and an optional additional-tags map, each with the default above.
2. **Set up shared locals and discover the zones.** Add a data source that lists the region's available, no-opt-in zones, and locals for the cluster name, the first two of those zones, and the base tags merged with the additional-tags variable.
3. **Wire the provider to your inputs.** Extend the existing AWS provider to apply the merged tags as default tags on everything it creates. The region comes from `AWS_REGION` in the environment, so the provider does not set it.
4. **Create the VPC.** Call the community VPC module with EKS-appropriate networking: the two discovered zones, one public and one private subnet per zone derived from the CIDR, a single NAT gateway, and the two subnet tags above.
5. **Initialize, plan, and apply.** A new module means re-running init. Review the plan, confirm it creates the VPC, subnets, gateways, and route tables, then apply.

## Done when

- `terraform apply` completes, creating a VPC, four subnets across two availability zones, an internet gateway, a single NAT gateway, and their route tables.
- `terraform validate` reports the configuration is valid.
- The public subnets carry the internet-facing load balancer tag and the private subnets carry the internal one.

## Note

The NAT gateway is the first resource in this course that bills continuously: a per-hour charge plus a per-gigabyte charge for the traffic it processes. From here on, build the habit of tearing the project down at the end of a working session. The remote state bucket is negligible to leave in place; the NAT gateway is not.
