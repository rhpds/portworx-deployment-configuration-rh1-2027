# Module 03 — Live Migrations and HA

### Brief Overview

Students write a persistence marker to a running VM, perform a live compute migration between worker nodes, and then simulate an unplanned node failure by rebooting the node hosting the VM. They observe STORK directing the scheduler to restart the VM on a node that already holds a Portworx replica, verify data persistence throughout, and wait for the cluster to fully recover before moving on.

### Audience and Time

- **Audience:** Red Hat Solutions Engineers, Architects, and field AEs
- **Estimated duration:** 20 minutes
- **Prerequisites for this module:** Module 02 complete; `centos-stream9-example` VM running in `vmtest`

### Learning Objectives

- Demonstrate live VM migration between compute nodes with no downtime using the OpenShift Console Actions menu
- Verify that Portworx RWX block storage retains guest data across a live migration
- Simulate an unplanned node failure with `oc debug node` and observe STORK-guided VM recovery on a surviving node
- Confirm data persistence after an HA restart and wait for all Portworx nodes to come back online

### Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Write persistence marker to the VM | 2 min |
| 2 | Live migrate the VM via the OpenShift Console | 5 min |
| 3 | Verify data after migration | 2 min |
| 4 | Reboot the VM's node and observe HA recovery | 8 min |
| 5 | Verify data and wait for cluster recovery | 3 min |

### Detailed Steps

1. Write `Hello from Red Hat and Portworx` to `~/text` inside the VM via `virtctl ssh -c`.
2. In the OpenShift Console, navigate to the VM's Overview tab, note its current node, then use Actions > Migration > Compute to trigger live migration.
3. After migration completes, SSH to the VM and confirm `cat ~/text` still returns the marker.
4. Find the VM's node with `oc get vmi centos-stream9-example -n vmtest -o jsonpath='{.status.nodeName}'`, then `oc debug node/$NODENAME`, chroot to host, and run `reboot`.
5. Watch `oc get pods,vms -o wide -n vmtest` until a new virt-launcher pod starts on a surviving node.
6. Retry SSH in a loop until the VM boots, then `cat ~/text` to confirm data survived the HA restart.
7. Wait for the rebooted node to rejoin and all three Portworx nodes to come online using the pxctl alias.
