# Module 06 — Manual Pool Relocate

### Brief Overview

Manual pool relocate is a Day-2 operation that moves data from one storage pool to another — used when a storage tier needs to be decommissioned, rebalanced, or upgraded. This module gives participants hands-on experience with the pxctl pool relocation workflow and demonstrates how Portworx maintains data integrity throughout the move. While auto-rebalance handles routine balancing (covered in Module 7's discussion), manual relocate gives operators explicit control over which data moves where and when. Completing this module rounds out the Day-2 operations story for the vSAN replacement narrative.

### Audience and Time

- **Audience:** Red Hat Solutions Engineers, Architects, and Account Executives; Modules 02-05 completion recommended (Portworx running with volumes present)
- **Estimated duration:** 15 minutes
- **Prerequisites for this module:** Portworx Enterprise online with at least one active volume in a storage pool; node restored to Online state after Module 05 failover

### Learning Objectives

- Execute a manual pool relocate operation using pxctl to move volume data from one storage pool to another
- Monitor the data movement progress and verify data integrity after the relocation completes

### Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Identify source and target pools | 3 min |
| 2 | Initiate the pool relocation | 4 min |
| 3 | Monitor relocation progress | 5 min |
| 4 | Verify data integrity post-relocation | 3 min |

### Detailed Steps

1. In the Showroom terminal, list all storage pools: `pxctl storage list`. Identify the pool IDs associated with the three worker nodes. Note the used and available capacity on each pool.
2. List all volumes: `pxctl volume list`. Identify the volume ID of the VM-backing volume from Module 04-05. Note which pool it is currently assigned to (run `pxctl volume inspect <volume-id>` and check the "Pool" field).
3. Identify the target pool for relocation: select a pool on a different node than the current primary replica. Note its pool ID.
4. Initiate the pool relocation using pxctl: `pxctl service pool-update --action move --pool <source-pool-id> --dest-pool <target-pool-id>`. Confirm the operation when prompted.
5. Monitor relocation progress: run `pxctl service pool-update --status` repeatedly (or `watch pxctl service pool-update --status`) until the operation reaches 100% completion. Note the estimated time to completion displayed by pxctl.
6. While the relocation is in progress, confirm that the VM from Module 04-05 remains Running: `oc get vm -n demo`. The relocation is non-disruptive to running workloads.
7. After the relocation completes, run `pxctl volume inspect <volume-id>` again. Confirm the volume replicas are now distributed across the expected nodes, including the target pool.
8. Run `pxctl status` and confirm all nodes are Online and the pool utilization reflects the data movement (source pool shows less used space; target pool shows more).
9. In the Portworx console plugin in the OCP web console, verify the volume's replica distribution matches the pxctl output.
10. Confirm data integrity: connect to the VM console (Virtualization > VirtualMachines > vm-demo > Console) and verify the OS is still responsive and file data is intact.

### Key Takeaways

- Manual pool relocate is a non-disruptive operation — workloads continue running while data moves between pools
- pxctl provides real-time progress visibility for pool relocations, giving operators confidence during the operation
- After relocation, Portworx verifies data integrity automatically before marking the operation complete
- Manual relocate is the operator's tool for explicit data placement; Module 7 explains when to use auto-rebalance instead

### Infrastructure Notes

- With only NVMe devices in the lab cluster, the relocation moves data between nodes (same device class) rather than between device tiers
- Relocation duration depends on volume size and cluster load; for the lab's small test volume this should complete in 1-2 minutes
- The `pxctl service pool-update` syntax may vary by PXE version; verify against PXE 3.7 documentation before writing module content
- If the volume is too small to demonstrate meaningful progress, the writer should stage a larger volume (5-10 GiB) in the pre-staged lab files
