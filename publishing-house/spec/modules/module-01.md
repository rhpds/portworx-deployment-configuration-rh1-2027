# Module 01 — Install Portworx

### Brief Overview

Students install Portworx Enterprise 3.7 on an existing OpenShift 4.22 cluster from scratch. They enable user workload monitoring, deploy the Portworx Operator from the OpenShift Software Catalog, create a StorageCluster CR via the CLI, verify the cluster with pxctl, and then deploy OpenShift Virtualization by creating the HyperConverged object. The module ends by reviewing the Portworx StorageClasses and StorageProfiles created for VM workloads.

### Audience and Time

- **Audience:** Red Hat Solutions Engineers, Architects, and field AEs; no prior Portworx knowledge required
- **Estimated duration:** 15 minutes
- **Prerequisites for this module:** Showroom terminal with cluster-admin oc access; OpenShift web console reachable

### Learning Objectives

- Enable OpenShift user workload monitoring so that Portworx can export metrics to the OpenShift monitoring stack
- Install the Portworx Enterprise Operator from the OpenShift Software Catalog and create a StorageCluster CR from the CLI
- Verify a healthy 3-node Portworx cluster using pxctl and the Portworx console plugin
- Deploy OpenShift Virtualization by creating the HyperConverged CR and confirm that Portworx StorageClasses and StorageProfiles for VM workloads are available

### Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Project setup and cluster orientation | 2 min |
| 2 | Enable user workload monitoring | 2 min |
| 3 | Install Portworx Operator from Software Catalog | 4 min |
| 4 | Create StorageCluster CR and verify with pxctl | 5 min |
| 5 | Install HyperConverged and review StorageClasses | 2 min |

### Detailed Steps

1. Create the `vmtest` project with `oc new-project vmtest` and explore cluster nodes with `oc get nodes`.
2. Create the `cluster-monitoring-config` ConfigMap in `openshift-monitoring` to enable user workload monitoring.
3. Navigate to Ecosystem > Software Catalog in the OpenShift Console, search for Portworx Enterprise, install the operator into a new `portworx` project, and enable the Console plugin.
4. Look up the Thanos Querier route, write the StorageCluster CR to `~/px-spec.yaml` using a here-doc, and apply it with `oc apply -f`.
5. Watch `px-cluster` pods with `watch oc -n portworx get pods -l name=portworx` until all three show `1/1 Running`.
6. Run `pxctl status --color` via the pxctl alias shortcut to confirm all 3 storage nodes are online.
7. Apply the HyperConverged CR with `scratchSpaceStorageClass: px-cdi-scratch` and `vmStateStorageClass: px-rwx-file-kubevirt`.
8. Inspect `oc get sc` and `oc get storageprofile px-rwx-block-kubevirt` to confirm Portworx VM StorageClasses are present.
9. Confirm boot image PVCs in `openshift-virtualization-os-images` are Bound on `px-rwx-block-kubevirt`.
10. Install `virtctl` from the cluster's ConsoleCLIDownload.
