# Module 02 — Installing Portworx Enterprise on OpenShift

### Brief Overview

This is the core installation module. Participants go from a bare OCP cluster with no storage operator to a fully operational Portworx Enterprise distributed storage layer in a single, instructor-guided session. The workflow mirrors exactly what a customer's Day-1 operator would do: generate a spec in the Portworx Central SaaS portal, pull the Portworx Operator from the OpenShift Software Catalog (OperatorHub), apply a StorageCluster custom resource, and validate the deployment with pxctl. Completing this module gives participants the muscle memory to guide a customer through the same process.

### Audience and Time

- **Audience:** Red Hat Solutions Engineers, Architects, and Account Executives; Module 01 completion assumed
- **Estimated duration:** 35 minutes
- **Prerequisites for this module:** Cluster-admin access confirmed (Module 01); Portworx Central credentials or pre-generated spec token provided in Showroom

### Learning Objectives

- Generate a storage spec in Portworx Central scoped to OCP 4.22 and Portworx Enterprise 3.7
- Deploy the Portworx Enterprise Operator from the OpenShift Software Catalog using the web console
- Create a StorageCluster custom resource from the generated spec and verify that distributed storage is online across all worker nodes using pxctl

### Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Generate a storage spec in Portworx Central | 8 min |
| 2 | Deploy the Portworx Operator from OperatorHub | 7 min |
| 3 | Create the StorageCluster custom resource | 10 min |
| 4 | Verify deployment with pxctl | 5 min |
| 5 | Identify the KDS vs. PXE install distinction | 5 min |

### Detailed Steps

1. Open the Portworx Central URL provided in Showroom. Log in with the credentials pre-staged in the Showroom terminal environment variable `PX_CENTRAL_TOKEN`.
2. In Portworx Central, navigate to Generate Spec. Select platform: OpenShift. Select Portworx version: 3.7. Select KVDB: Internal (uses the dedicated KVDB NVMe device on each worker). Select storage type: Auto (detects NVMe devices automatically). Download or copy the generated `StorageCluster` YAML manifest.
3. In the OpenShift web console, navigate to Operators > OperatorHub. Search for "Portworx". Select "Portworx Enterprise" from Pure Storage. Click Install.
4. On the Install Operator page, set the update channel to the 3.7 channel. Set namespace to `kube-system`. Leave all other defaults. Click Install.
5. Monitor the operator installation under Operators > Installed Operators until the status shows "Succeeded". This takes approximately 2-3 minutes.
6. Navigate to the installed Portworx Enterprise operator. Click "Create StorageCluster". Switch to the YAML view.
7. Paste the generated StorageCluster YAML from Portworx Central. Confirm the KVDB device (`/dev/nvme2n1`) and data device (`/dev/nvme1n1`) paths match the lab cluster's worker node device layout.
8. Click Create. Monitor the StorageCluster status in the Operators > Installed Operators > Portworx Enterprise view.
9. In the Showroom terminal, run `oc get storagecluster -n kube-system` and wait for phase: Online.
10. Run `oc get pods -n kube-system | grep portworx` and confirm one portworx pod per worker node is Running.
11. Run `pxctl status` from within a portworx pod: `kubectl exec -it -n kube-system $(kubectl get pods -n kube-system -l name=portworx -o jsonpath='{.items[0].metadata.name}') -- /opt/pwx/bin/pxctl status`. Confirm: 3 nodes Up, storage capacity ~150 GiB total.
12. Note the key distinction between KDS and PXE: the PXE StorageCluster enables enterprise services (CloudSnap, PX-AutoPilot, PX-Security) by default; KDS omits these in its simplified profile.

### Key Takeaways

- Portworx Central generates a validated StorageCluster spec — customers do not hand-craft the YAML
- The Portworx Operator is available in the OpenShift Software Catalog, making installation a standard OCP operator workflow
- pxctl is the primary CLI for validating storage health; the three-node output confirms distributed storage is operational
- The KDS installation path uses a simplified StorageCluster profile — PXE adds enterprise features on top of the same operator framework

### Infrastructure Notes

- Each worker has: `/dev/nvme1n1` (50 GiB data device), `/dev/nvme2n1` (32 GiB KVDB device)
- Total usable Portworx capacity: approximately 150 GiB across 3 storage nodes
- Portworx Central credentials are pre-staged; the spec token must be scoped to PXE 3.7 + OCP 4.22 (coordinated with Chris Crow at Pure Storage before the event)
- Operator installation may take 3-5 minutes in a shared environment; StorageCluster reconciliation may take 5-10 minutes
