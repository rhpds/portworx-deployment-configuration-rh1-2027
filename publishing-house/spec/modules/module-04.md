# Module 04 — Virtual Machine Snapshots

### Brief Overview

Students take a Portworx-backed snapshot of a running VM, simulate accidental data loss by deleting `/etc/fstab`, stop the VM, restore it from the snapshot via the OpenShift Console, restart the VM, and verify the file is recovered. The module demonstrates Portworx CSI snapshot integration with OpenShift Virtualization.

### Audience and Time

- **Audience:** Red Hat Solutions Engineers, Architects, and field AEs
- **Estimated duration:** 20 minutes
- **Prerequisites for this module:** Module 03 complete; `centos-stream9-example` VM running in `vmtest`

### Learning Objectives

- Take a VM snapshot from the OpenShift Virtualization Console Snapshots tab
- Simulate destructive data loss by deleting a critical guest file via virtctl ssh
- Stop a VM, restore it from a snapshot, and restart it using the Console
- Verify that the restored guest filesystem matches the pre-snapshot state

### Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Take a VM snapshot | 4 min |
| 2 | Delete /etc/fstab inside the VM | 3 min |
| 3 | Stop VM and restore snapshot | 8 min |
| 4 | Restart VM and verify restore | 5 min |

### Detailed Steps

1. In the Console, navigate to Virtualization > VirtualMachines > centos-stream9-example > Snapshots tab, click Take Snapshot, and accept the warning that cloudinitdisk is excluded.
2. SSH into the VM and run `sudo rm /etc/fstab`, then confirm the file is gone with `cat /etc/fstab`.
3. In the Console, use Actions > Stop to shut down the VM.
4. On the Snapshots tab, open the kebab menu for the snapshot and select Restore VirtualMachine from snapshot; confirm the restore.
5. Start the VM again via Actions > Start and wait for it to reach Running status.
6. SSH into the VM and run `cat /etc/fstab` to confirm the file has been restored.

### Key Takeaways

- Portworx snapshots are storage-consistent, crash-consistent point-in-time copies that do not require the VM to be powered off
- Restore creates a new PVC from the snapshot; the VM is then restarted against the restored disk — fstab content confirms filesystem integrity
- Snapshot and restore operations are native to OCP Virtualization; no additional tooling or agents inside the guest are required
