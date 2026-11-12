# Module 05 — Autopilot

### Brief Overview

Students configure Portworx Autopilot to automatically expand a VM's data disk when usage exceeds 30%. They create a second VM with a 5 GiB data disk, write an AutopilotRule, label the namespace and PVC, fill the disk past the threshold, and observe Autopilot expand the PVC to 10 GiB with no manual intervention. They then grow the guest partition and filesystem inside the VM to use the new space.

### Audience and Time

- **Audience:** Red Hat Solutions Engineers, Architects, and field AEs
- **Estimated duration:** 20 minutes
- **Prerequisites for this module:** Module 04 complete; Autopilot pod running in `portworx` namespace; Thanos Querier URL correctly set in StorageCluster

### Learning Objectives

- Create an AutopilotRule that monitors Portworx PVC usage and doubles volume size when usage exceeds 30%, up to a 20 GiB maximum
- Label a namespace and PVC to bring them into scope of an Autopilot rule
- Observe Autopilot state transitions (Normal → Triggered → ActiveActionsInProgress → ActiveActionsTaken) in Kubernetes events
- Grow the guest partition and filesystem inside a VM to consume the automatically expanded PVC

### Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Create the Autopilot VM with a 5 GiB data disk | 4 min |
| 2 | Create the AutopilotRule and label namespace/PVC | 3 min |
| 3 | Format and mount the data disk inside the VM | 3 min |
| 4 | Fill the disk and watch Autopilot respond | 7 min |
| 5 | Expand the guest partition and verify free space | 3 min |

### Detailed Steps

1. Apply a VirtualMachine manifest for `centos-stream9-autopilot` with two DataVolumeTemplates: a 30 GiB boot disk and a 5 GiB blank data disk (`centos-stream9-autopilot-data-disk`) on `px-rwx-block-kubevirt`.
2. Apply the `volume-resize` AutopilotRule: selector `expand=true`, condition `px_volume_usage_bytes/px_volume_capacity_bytes > 30%` for 5 minutes, action resize by 100%, maxsize 20 GiB.
3. Label namespace `vmtest` and PVC `centos-stream9-autopilot-data-disk` with `expand=true`.
4. SSH into the VM and format `/dev/vdb` with fdisk + mkfs.ext4, then mount at `/data`.
5. Run `sudo shred -n 1 -s 4G /data/file` to fill the disk past the 30% threshold.
6. Watch `oc get events --field-selector involvedObject.kind=AutopilotRule` until `ActiveActionsTaken` appears.
7. Confirm `oc get pvc -n vmtest` shows the data disk expanded to 10 GiB.
8. Run `parted` and `resize2fs` inside the VM to grow the partition and filesystem to fill the new capacity.
9. Check `df -h` inside the VM to confirm `/data` shows the expanded size.

### Key Takeaways

- Autopilot monitors Prometheus metrics and expands PVCs automatically when usage exceeds a configured threshold — no manual intervention required
- The AutopilotRule CRD uses label selectors on both the namespace and the PVC; only labelled resources are managed
- After Autopilot expands the PVC, the guest filesystem must be grown manually with parted and resize2fs — Portworx resizes the block device but does not modify the partition table
