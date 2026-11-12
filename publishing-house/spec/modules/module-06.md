# Module 06 — Dynamic App I/O Control

### Brief Overview

Students use Portworx Dynamic Application I/O Control to tie a volume's IOPS limit to its size. They create a StorageClass with an IOPS scale factor, deploy an fio benchmark workload on a 10 GiB PVC, observe the 500 IOPS ceiling, expand the PVC to 20 GiB and watch Autopilot raise the limit to 1000, expand again to 40 GiB and see the limit clamped at the 1500 ceiling, opt out of dynamic limits for a single PVC, and finally clean up. The module ends with a lab summary.

### Audience and Time

- **Audience:** Red Hat Solutions Engineers, Architects, and field AEs
- **Estimated duration:** 20 minutes
- **Prerequisites for this module:** Module 05 complete; Portworx Autopilot running

### Learning Objectives

- Create a Portworx StorageClass with dynamic I/O throttle parameters (scale factor, floor, and ceiling for IOPS and bandwidth)
- Deploy a containerized fio benchmark and confirm the IOPS limit matches the StorageClass scale formula for the current volume size
- Expand a PVC and observe Autopilot automatically recalculate and raise the IOPS limit without restarting the workload
- Opt out of dynamic I/O control for a single PVC using an annotation and remove limits manually with pxctl

### Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Create the px-dynamic-io StorageClass | 3 min |
| 2 | Deploy fio workload on a 10 GiB PVC and verify 500 IOPS limit | 5 min |
| 3 | Expand PVC to 20 GiB and observe 1000 IOPS | 4 min |
| 4 | Expand PVC to 40 GiB and observe 1500 IOPS ceiling | 3 min |
| 5 | Opt out and remove limits; clean up | 3 min |
| 6 | Lab summary | 2 min |

### Detailed Steps

1. Apply a `px-dynamic-io` StorageClass with `io_throttle_iops_scale_factor: "50"`, floor 300, ceiling 1500, and `allowVolumeExpansion: true`.
2. Create the `appio` project and deploy a 10 GiB PVC + fio Deployment that writes a 6 GiB file then continuously runs 4k random reads.
3. Save the PV name in `VOL` and run `pxctl volume inspect ${VOL}` to confirm Max Read/Write IOPS = 500.
4. Follow fio logs and confirm measured IOPS stay at or below 500.
5. Patch the PVC to 20 GiB; after Autopilot detects the new size, confirm limits rise to 1000 IOPS without restarting the container.
6. Patch PVC to 40 GiB; confirm limits clamp to the 1500 ceiling even though `40 × 50 = 2000`.
7. Annotate the PVC with `portworx.io/throttle-auto-update=false` to opt out, then run `pxctl volume update --max_iops off,off ${VOL}` to remove limits entirely.
8. Delete the `appio` project and `px-dynamic-io` StorageClass.
