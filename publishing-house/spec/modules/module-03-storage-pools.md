# Module 03 — Storage and Dynamic Storage Pools

### Brief Overview

With Portworx Enterprise running, this module teaches participants how storage is organized and exposed to OpenShift workloads. Portworx aggregates raw block devices into storage pools, and pools back StorageClasses that Kubernetes workloads consume. This module walks through pool inspection, StorageClass creation, and the dynamic pool provisioning model — including how a single Portworx cluster can serve multiple back-end device types (SSD, HDD, NVMe) through separate pools, each with its own StorageClass and performance profile. This is the foundation participants need before provisioning VM volumes in Module 4.

### Audience and Time

- **Audience:** Red Hat Solutions Engineers, Architects, and Account Executives; Module 02 completion required (Portworx must be online)
- **Estimated duration:** 20 minutes
- **Prerequisites for this module:** Portworx Enterprise deployed and in Online phase (confirmed by pxctl status in Module 02)

### Learning Objectives

- Inspect Portworx storage pools using pxctl and identify the mapping from raw block devices to pool capacity
- Create OpenShift StorageClasses backed by Portworx volumes with specific replication factor, access mode, and performance parameters
- Configure multiple storage pools for heterogeneous back-end types and verify that each pool maps to a distinct StorageClass

### Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Portworx storage concepts: volumes, pools, StorageClasses | 4 min |
| 2 | Inspect existing pools with pxctl | 4 min |
| 3 | Create custom StorageClasses | 6 min |
| 4 | Verify PVC binding against pools | 6 min |

### Detailed Steps

1. In the Showroom terminal, run `pxctl volume list` to see the current volume inventory (should be empty — no workloads yet).
2. Run `pxctl storage list` to display the pool inventory. Note the pool ID, device path (`nvme1n1`), size, and status for each of the three worker nodes.
3. Run `pxctl status` and locate the "Storage Pool" section. Identify the total pool size, used space, and replication state.
4. In the OpenShift console, navigate to Storage > StorageClasses. Review any StorageClasses that Portworx created automatically during operator installation.
5. In the Showroom terminal, create a high-performance StorageClass for RWO (single-node) workloads. Apply the following YAML from the pre-staged file `~/lab-files/sc-portworx-rwx.yaml`:
   ```yaml
   kind: StorageClass
   apiVersion: storage.k8s.io/v1
   metadata:
     name: portworx-rwx
   provisioner: kubernetes.io/portworx-volume
   parameters:
     repl: "2"
     io_profile: "db"
     sharedv4: "true"
   volumeBindingMode: Immediate
   ```
6. Apply the StorageClass: `oc apply -f ~/lab-files/sc-portworx-rwx.yaml`. Verify it appears in `oc get sc`.
7. Create a test PVC against the new StorageClass using the pre-staged `~/lab-files/pvc-test.yaml`. Apply it: `oc apply -f ~/lab-files/pvc-test.yaml -n default`.
8. Run `oc get pvc -n default` and verify the PVC binds (status: Bound).
9. Run `pxctl volume list` again and confirm a new volume appears, mapped to the pool on the worker nodes.
10. Delete the test PVC: `oc delete pvc test-pvc -n default`. Confirm the volume is reclaimed in `pxctl volume list`.
11. Instructor note: describe how adding SSD vs. NVMe devices to the StorageCluster spec would create separate pools, each mapped to a StorageClass with a distinct `io_profile` — participants will see this in action in a production environment.

### Key Takeaways

- Portworx pools are the bridge between raw block devices and Kubernetes StorageClasses — every PVC is backed by a volume in a named pool
- The replication factor (`repl: 2`) controls how many nodes hold a copy of the data; this is the HA knob for storage
- Dynamic provisioning means a PVC binding triggers automatic volume creation — no pre-provisioning required
- Multiple back-end types can coexist in a single Portworx cluster; the StorageClass `io_profile` selects the right pool

### Infrastructure Notes

- Lab cluster has only NVMe devices; multi-back-end pool demo is a discussion in Module 7, not hands-on
- The `sharedv4: "true"` parameter enables RWX access mode (required for VM live migration in Module 4)
- Pre-staged YAML files are in `~/lab-files/` on the Showroom terminal
