# Portworx Enterprise Deployment and Configuration on OpenShift

<!-- Design document for the RH1 2027 lab. Source of truth: RH1 2027 lab description. -->

## Overview

This lab teaches Red Hat field engineers how to install and operate Portworx Enterprise on OpenShift as a production replacement for VMware vSAN. The context is the VMware-to-OpenShift migration wave: 86% of organizations are already shrinking their VMware footprint, but only 4% have finished, and these deals stall because no one can answer what replaces vSAN and the datastore.

Participants install Portworx Enterprise from scratch on a dedicated OCP cluster. They generate a storage spec in Portworx Central, deploy the Portworx Operator from the OpenShift Software Catalog, create a StorageCluster custom resource, and verify with pxctl that a distributed storage layer is online across all nodes. They then configure dynamic storage pools, deploy VM workloads on RWX block storage, simulate a node failure and verify workload continuity, and execute a manual pool relocate — completing the full operator-day-1 and day-2 loop in under two hours.

## Target Audience

- **Role:** Red Hat Solutions Engineers, Architects, and Account Executives in field roles
- **Experience level:** Intermediate
- **What they already know:** Basic OpenShift navigation (OperatorHub, console, oc CLI); familiarity with persistent storage concepts (PVCs, StorageClasses); awareness of VMware vSAN at a conceptual level
- **What they don't know:** Portworx Enterprise architecture and installation, Portworx Central spec generation, distributed storage pools, per-VM storage telemetry via pxctl

## Prerequisites

- Familiarity with the OpenShift web console and oc CLI at a basic level
- Conceptual understanding of Kubernetes PersistentVolumes and StorageClasses
- No prior Portworx knowledge required; no storage array or advanced CLI expertise required
- Prerequisites are assumed and cannot be auto-validated by the lab environment

## Learning Objectives

1. Install Portworx Enterprise on OpenShift by generating a spec in Portworx Central, deploying the Portworx Operator from the OpenShift Software Catalog, and creating a StorageCluster custom resource
2. Configure dynamic storage pools and OpenShift StorageClasses backed by Portworx distributed volumes, verifying pool health with pxctl
3. Deploy VM workloads on RWX block storage, enable the Portworx console plugin, and observe enterprise data services as a native tab in the OpenShift console
4. Execute a storage-level failover, verify workload continuity after a simulated node failure, and analyze per-VM storage telemetry
5. Perform a manual pool relocate operation and verify data integrity across storage tiers

## Content Type

Lab (hands-on)

## Products & Technologies

- Red Hat OpenShift Container Platform 4.22
- Red Hat OpenShift Virtualization (for VM workload modules)
- Portworx Enterprise 3.7 (Pure Storage partner technology — not a Red Hat product)
- Portworx Central (spec generation portal — SaaS, partner-managed)
- pxctl (Portworx CLI, included with Portworx Enterprise)

## Module Map

| Module | Title | Duration |
|--------|-------|----------|
| 1 | Overview of Kube Datastore | 15 min |
| 2 | Installing Portworx Enterprise on OpenShift | 35 min |
| 3 | Storage and Dynamic Storage Pools | 20 min |
| 4 | Deploying Demo Workloads | 20 min |
| 5 | Failover Exercise | 20 min |
| 6 | Manual Pool Relocate | 15 min |
| 7 | Discussion: Auto-Rebalancing and Multiple Back-ends | 15 min |
| — | **Total hands-on (Modules 1-6)** | **2 hours 5 min** |
| — | Facilitated discussion (Module 7) | 15 min |
| — | **Total lab** | **~2 hours 20 min** |

<!-- Module 7 is a facilitated discussion with no hands-on steps. -->

## Difficulty Level

Intermediate

## Environment

**Learner view:** Each student receives a dedicated multi-node OpenShift 4.22 cluster. The cluster is provisioned with three control plane nodes and three worker nodes; each worker has two NVMe block devices (one for Portworx data, one for KVDB). OpenShift Virtualization is pre-installed. Portworx Enterprise is NOT pre-installed — students install it during Module 2 as the core lab activity. The OpenShift web console and a Showroom terminal with oc and pxctl access are available from the first module.

**Automation needed:** Yes

The lab provisioning automation must:
- Provision a per-student multinode OCP 4.22 cluster (3 control plane, 3 workers) with NVMe block devices attached and formatted for Portworx use
- Pre-install Red Hat OpenShift Virtualization operator but leave Portworx Enterprise NOT installed (students install it)
- Configure an oc login session and terminal with cluster-admin credentials in Showroom
- Stage sample VM manifests and workload YAMLs in the student's home directory for Modules 4-6
- Provide Portworx Central credentials or a pre-generated spec token scoped to OCP 4.22 + PXE 3.7 (coordinated with Pure Storage)

## Infrastructure Requirements

- **Cloud provider:** TBD — confirmed in infrastructure phase
- **Cluster type:** TBD — confirmed in infrastructure phase
- **OCP version:** TBD — confirmed in infrastructure phase
- **Topology:** TBD — confirmed in infrastructure phase
- **Sizing:** TBD — confirmed in infrastructure phase
- **Automation approach:** TBD — confirmed in infrastructure phase
- **AI/MaaS:** TBD — confirmed in infrastructure phase
- **External services:** TBD — confirmed in infrastructure phase
- **AAP version:** TBD — confirmed in infrastructure phase
- **Non-GA products:** TBD — confirmed in infrastructure phase
