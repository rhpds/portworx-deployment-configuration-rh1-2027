# Module 05 — Failover Exercise

### Brief Overview

This module is the proof-of-concept moment for the vSAN replacement story. Participants simulate a worker node failure with a VM actively running on Portworx RWX block storage, observe Portworx's automatic storage-level failover, and verify that the workload continues without manual intervention. The module also surfaces per-VM storage telemetry during and after the failure event — showing not just that the VM survived, but exactly what happened at the storage layer. This is the scenario most likely to appear in a customer proof-of-concept evaluation.

### Audience and Time

- **Audience:** Red Hat Solutions Engineers, Architects, and Account Executives; Module 04 completion required (VM must be running on Portworx RWX storage)
- **Estimated duration:** 20 minutes
- **Prerequisites for this module:** VM from Module 04 must be in Running state; Portworx console plugin enabled

### Learning Objectives

- Simulate a storage node failure by cordoning and draining a worker node while a VM is running
- Verify that Portworx performs a storage-level failover and the VM workload continues without data loss
- Analyze per-VM storage telemetry before, during, and after the failover event using pxctl and the Portworx console plugin

### Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Verify pre-failover baseline (VM running, telemetry normal) | 4 min |
| 2 | Simulate node failure (cordon + drain) | 5 min |
| 3 | Observe Portworx failover in progress | 5 min |
| 4 | Verify workload continuity and restore node | 6 min |

### Detailed Steps

1. In the Showroom terminal, confirm the VM from Module 04 is still Running: `oc get vm -n demo`.
2. Identify which worker node the VM's volume is attached to: `pxctl volume inspect <volume-id>` — note the "Attached On" field.
3. In the Portworx console plugin in the OCP web console, observe the current I/O telemetry for the VM volume (note IOPS and replica count = 2).
4. In the Showroom terminal, cordon the node that holds the VM volume attachment: `oc adm cordon <node-name>`. This prevents new pods from scheduling but does not drain existing ones yet.
5. Drain the node: `oc adm drain <node-name> --ignore-daemonsets --delete-emptydir-data`. This evicts running pods on the node.
6. Watch the VM status: `oc get vm -n demo -w`. The VM may briefly transition through a migration phase as OpenShift Virtualization detects the node drain.
7. In the Showroom terminal, watch Portworx respond to the lost node: `pxctl status` — observe that the affected node transitions to Offline state.
8. Run `pxctl volume inspect <volume-id>` again. Observe that the "Attached On" field has changed to a healthy node and the replication status shows re-replication in progress.
9. In the Portworx console plugin, observe the telemetry timeline showing the failover event: a brief I/O pause followed by resumption on the new node.
10. Confirm the VM is still Running (or has restarted and is Running again): `oc get vm -n demo`.
11. Test workload continuity by connecting to the VM console in the OCP web console (Virtualization > VirtualMachines > vm-demo > Console tab). Verify the OS is responsive.
12. Restore the node: `oc adm uncordon <node-name>`. Run `pxctl status` and confirm the node returns to Online state and replication rebuilds to the target replica count.

### Key Takeaways

- Portworx performs automatic storage-level failover when a node goes offline — no manual operator intervention is required
- The replication factor set during StorageClass creation (Module 03) determines how many node failures can be tolerated without data loss
- Per-VM telemetry shows the exact moment of failover — customers can use this for SLA audit and incident reporting
- The failed node returns to full replication state automatically after uncordoning — no manual data repair steps needed

### Infrastructure Notes

- The cordon + drain pattern is the standard simulation for a node failure in a lab environment; it avoids actually losing the VM (destructive options like `poweroff` the hypervisor are not available in the CNV lab environment)
- Draining may trigger OpenShift Virtualization LiveMigration if the VM is migratable — this is expected and reinforces the RWX storage story
- Re-replication after node uncordon can take several minutes depending on volume size; the module wraps before full rebuild completes
- If the VM fails to restart automatically, `oc start vm vm-demo -n demo` restarts it manually
