# Lab: Add the Networking Addons

## Goal

Install the two core addons that a node needs from the moment it joins: pod networking and service routing. By the end both are managed by EKS and active on the cluster, even though it still has no nodes. Getting them in place first is what lets your nodes come up healthy when you add them next instead of hanging.

## Starting point

You continue from the applied project: the inputs, locals, provider, VPC, both IAM roles, and the active control plane are in place. You add one new file for the addons alongside them.

## Configuration to target

- **New file:** `infra/addons.tf` (or just add them to `infra/cluster.tf`).
- **Pod networking addon:** the `vpc-cni` addon, installed on the existing cluster. Leave the version unset so EKS installs the default for the cluster's Kubernetes version.
- **Service routing addon:** the `kube-proxy` addon, on the same cluster, version unset for the same reason.
- **Conflict handling:** set both addons to overwrite on create and on update, so EKS owns their configuration cleanly.
- **Tags:** the project base tags.

## Tasks

1. **Create the addons file with pod networking.** Add the `vpc-cni` addon on the cluster, with the version left to EKS and conflicts set to overwrite.
2. **Add service routing.** Add the `kube-proxy` addon in the same file, configured the same way.
3. **Apply.** Review the plan (two addon resources) and apply. Terraform waits for each addon to become active.

## Done when

- `terraform apply` completes, creating both addons.
- `terraform validate` reports the configuration is valid.
- Listing the cluster's addons shows `vpc-cni` and `kube-proxy`, and each reports status `ACTIVE`.

## Note

The addons install and go active with no nodes present. That is intentional: pod networking and service routing have to be ready before a node joins, or the node registers and then sits `NotReady` with nothing able to schedule. The addons themselves are not billed; nodes are the next cost, added when you add worker capacity.
