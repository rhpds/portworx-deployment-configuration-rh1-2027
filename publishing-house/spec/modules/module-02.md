# Module 02 — Create a New VM

### Brief Overview

Students provision a CentOS Stream 9 virtual machine on OpenShift Virtualization backed by Portworx persistent storage. They first create the VM using the oc CLI, SSH into it via virtctl, and then optionally create a second VM through the OpenShift Console to explore the web UI workflow.

### Audience and Time

- **Audience:** Red Hat Solutions Engineers, Architects, and field AEs; no prior OCP Virtualization knowledge required
- **Estimated duration:** 25 minutes
- **Prerequisites for this module:** Module 01 complete; Portworx and HyperConverged running; `vmtest` project exists; virtctl installed

### Learning Objectives

- Generate an SSH keypair and store the public key as an OpenShift Secret for VM access via virtctl
- Create a VirtualMachine manifest that clones its boot disk from a Portworx-backed boot image using a DataVolumeTemplate with no explicit StorageClass
- SSH into a running VM through the OpenShift control plane using `virtctl ssh` without configuring VM networking
- Create a virtual machine through the OpenShift Virtualization web console and explore VM metrics, snapshots, and YAML tabs

### Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Generate SSH keypair and create authorized-keys Secret | 3 min |
| 2 | Create VM via oc CLI and verify boot | 12 min |
| 3 | SSH into the VM with virtctl | 3 min |
| 4 | Create VM via OpenShift Console (optional) | 7 min |

### Detailed Steps

1. Run `ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""` and create the `authorized-keys` Secret in the `vmtest` namespace.
2. Apply a VirtualMachine manifest for `centos-stream9-example` referencing the `centos-stream9` DataSource; no StorageClass specified — Portworx default `px-rwx-block-kubevirt` is used.
3. Watch `oc get vms,dv -n vmtest` until the VM reaches Running status.
4. SSH using `virtctl ssh cloud-user@vm/centos-stream9-example -n vmtest -i ~/.ssh/id_rsa`.
5. Navigate to Virtualization > VirtualMachines in the console, create a new VM, choose centos.stream9, and explore its Overview, Snapshots, and YAML tabs.
