# Module 04 — Deploying Demo Workloads

### Brief Overview

With storage configured, participants now put Portworx Enterprise to work with real OpenShift Virtualization VM workloads. This module bridges the storage configuration work from Modules 2-3 to the VM-centric story that resonates with VMware customers: VMs running on RWX block storage, enterprise data services visible in the OpenShift console, and per-VM storage telemetry that CSI drivers alone cannot provide. The Portworx console plugin is a key deliverable here — it transforms Portworx from an invisible infrastructure component into a visible, native OCP console feature.

### Audience and Time

- **Audience:** Red Hat Solutions Engineers, Architects, and Account Executives; Modules 01-03 completion required
- **Estimated duration:** 20 minutes
- **Prerequisites for this module:** Portworx Enterprise online; `portworx-rwx` StorageClass created and verified (Module 03); Red Hat OpenShift Virtualization operator pre-installed in the lab environment

### Learning Objectives

- Deploy a VM workload backed by Portworx RWX block storage using OpenShift Virtualization
- Enable the Portworx console plugin and verify that enterprise data services appear as a native tab in the OpenShift console
- Verify storage connectivity and observe per-VM storage telemetry via the Portworx console plugin and pxctl

### Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Deploy a VM on RWX block storage | 7 min |
| 2 | Enable the Portworx console plugin | 5 min |
| 3 | Observe enterprise data services in the OCP console | 4 min |
| 4 | Review per-VM storage telemetry | 4 min |

### Detailed Steps

1. In the OpenShift web console, navigate to Virtualization > VirtualMachines. Confirm that OpenShift Virtualization is installed (the Virtualization menu item is present).
2. In the Showroom terminal, apply the pre-staged VM manifest: `oc apply -f ~/lab-files/vm-demo.yaml -n demo`. This creates a VirtualMachine object backed by a DataVolume that uses the `portworx-rwx` StorageClass.
3. Run `oc get vm -n demo` and wait for the VM to reach Running phase.
4. Run `oc get pvc -n demo` and confirm the DataVolume PVC is Bound and shows the `portworx-rwx` StorageClass.
5. In the Showroom terminal, run `pxctl volume list` and locate the volume backing the VM's PVC. Note the volume ID, replication status, and attached node.
6. In the OpenShift web console, navigate to Operators > Installed Operators > Portworx Enterprise. Locate the "Console Plugin" section. Enable the Portworx console plugin by setting the plugin status to Enabled.
7. After the console refreshes (this may require a page reload), navigate to Storage in the left nav. Observe that a new "Portworx" tab or section appears alongside the native OCP storage views.
8. Click into the Portworx storage view. Observe the volume list showing the VM's backing volume, replication factor, I/O stats, and attached node.
9. Click on the volume backing the VM. Review the per-VM telemetry: IOPS, throughput, latency, replica locations.
10. Return to the Showroom terminal. Run `pxctl volume inspect <volume-id>` using the volume ID observed in step 5. Confirm the telemetry matches what the console plugin displays.
11. Navigate to Virtualization > VirtualMachines in the OCP console. Select the running VM. Note the storage section showing the Portworx-backed DataVolume.

### Key Takeaways

- Portworx RWX block storage enables VM workloads to run with shared-access semantics, which is required for live migration
- The Portworx console plugin makes enterprise storage telemetry a first-class citizen in the OpenShift console — no separate management UI required
- Per-VM storage telemetry (IOPS, throughput, latency, replica location) is only available with Portworx; standard CSI drivers do not expose this level of observability
- This is the demo scenario that closes the vSAN replacement conversation: the customer's VMs run on Kubernetes with enterprise storage telemetry and no storage array

### Infrastructure Notes

- OpenShift Virtualization operator must be pre-installed by lab automation; students should not install it during the lab (it adds 10+ minutes)
- The Portworx console plugin requires an OCP console restart after enabling; the Showroom tab may need a reload
- Pre-staged VM manifest (`~/lab-files/vm-demo.yaml`) should use a small Fedora or RHEL CoreOS image to minimize DataVolume import time
- Telemetry data takes 1-2 minutes to populate after the VM starts running I/O
