# Lab: Add the Spot Node Group and DNS

## Goal

Add worker capacity and in-cluster DNS, in that order. By the end you have a Spot managed node group whose nodes register and go `Ready`, and CoreDNS scheduled onto them. This is the step that turns a control plane with addons into a cluster that can actually run workloads.

## Starting point

You continue from the applied project: the inputs, locals, provider, VPC, both IAM roles, the active control plane, and the two networking addons are all in place. You add the node group and DNS alongside them.

**IMPORTANT:** Make sure to enable the NAT gateway before deploying the node group! Otherwise, EC2 instances will not start up.

## Configuration to target

- **New variables (in `infra/variables.tf`):** a list of node instance types (default several small and medium types across two families), and the desired, minimum, and maximum node counts (default 2, 1, and 3).
- **New file `infra/node_group.tf`:** a managed node group named from the cluster name with a `-default` suffix, using:
  - the node role from earlier and the VPC's private subnets,
  - Spot capacity across the instance-types list,
  - the desired, min, and max sizes from the new variables,
  - an update cap of one node unavailable at a time,
  - an explicit dependency on the node policy attachment and both networking addons, so nodes join into working networking,
  - a lifecycle rule that ignores later changes to the desired size.
- **CoreDNS (added to `infra/addons.tf` or `infra/cluster.tf`):** the `coredns` managed addon, with an explicit dependency on the node group so it installs only after there are `Ready` nodes to run on.

## Tasks

1. **Add the node group variables.** Add the instance-types list and the desired, min, and max size variables.
2. **Enable NAT Gateway.** Toggle the enable_outbound_internet_access to true!! Very important, otherwise EC2 instances will not start up.
3. **Write the node group.** Create the managed node group on Spot capacity, wired to the node role, the private subnets, and the new sizing variables, with the update cap, the dependency on the networking addons and node policy, and the desired-size lifecycle rule.
4. **Add the DNS addon after the node group.** Add `coredns` to the addons file with a dependency on the node group, so DNS installs only once nodes are `Ready`.
5. **Apply.** Review the plan (a node group and the CoreDNS addon) and apply. The node group takes a few minutes to provision and join.

## Done when

- `terraform apply` completes, creating the node group and the CoreDNS addon.
- `terraform validate` reports the configuration is valid.
- The node group reports status `ACTIVE` on Spot capacity, and CoreDNS reports status `ACTIVE`, meaning it found `Ready` nodes to schedule onto.

## Note

The order matters and is enforced in code. The node group depends on the networking addons, so nodes come up `Ready` instead of stalling. CoreDNS depends on the node group, so its pods have somewhere to land instead of sitting unscheduled. This is also where real cost begins: the nodes bill from the moment they launch, on top of the control plane and NAT gateway already running, so plan to tear the cluster down at the end of a session.
