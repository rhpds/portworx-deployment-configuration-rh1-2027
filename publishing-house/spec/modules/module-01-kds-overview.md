# Module 01 — Overview of Kube Datastore

### Brief Overview

Kube Datastore (KDS) is Red Hat's lightweight, OCP-native block storage layer bundled with OpenShift Virtualization. This module positions KDS relative to Portworx Enterprise so participants can answer the first question any customer asks: "Which storage product do I need?" The module grounds the conversation in real market data before participants touch the Portworx installer — ensuring they understand the business context (the VMware migration wave) and the technical scope of what they are about to deploy.

### Audience and Time

- **Audience:** Red Hat Solutions Engineers, Architects, and Account Executives; no prior Portworx knowledge required
- **Estimated duration:** 15 minutes
- **Prerequisites for this module:** Access to the Showroom terminal and OpenShift web console; cluster-admin credentials available

### Learning Objectives

- Compare Kube Datastore and Portworx Enterprise on capacity, topology support, and enterprise feature set to identify the correct product for a given customer scenario
- Articulate the vSAN replacement narrative using market data (45+ enterprises, 100,000+ VM volumes, 30-50% cost savings) to support a field conversation
- Identify the key distinction between a KDS deployment and a standard Portworx Enterprise install before beginning the installation module

### Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Market context: the VMware migration gap | 4 min |
| 2 | What is Kube Datastore (KDS)? | 4 min |
| 3 | KDS vs. Portworx Enterprise — decision matrix | 4 min |
| 4 | Lab environment orientation | 3 min |

### Detailed Steps

1. Read the opening scenario in Showroom: a VMware renewal landed on a customer's CFO's desk and the deal is stalling because nobody can answer what replaces vSAN.
2. Review the market statistics panel: 86% of organizations shrinking VMware footprint; only 4% finished; 45+ enterprises running VMs on Kubernetes; 100,000+ VM volumes in production; 30-50% savings vs. prior virtualization spend.
3. Navigate to the KDS product overview section. Read the definition: KDS is the Kubernetes-native block storage layer bundled with OpenShift Virtualization for lightweight VM storage scenarios.
4. Review the KDS vs. Portworx Enterprise decision matrix. Note the differentiation on: cluster scale, enterprise data services (encryption, DR, auto-rebalance), heterogeneous back-end support, and telemetry depth.
5. Identify the key installation distinction flagged in the matrix: a KDS deployment uses a simplified StorageCluster profile that omits the Portworx Enterprise feature set — students will deploy the full PXE profile in Module 2.
6. Open the OpenShift web console in the browser tab pre-loaded in Showroom. Confirm cluster-admin access by viewing the node list (Compute > Nodes). Verify three worker nodes are visible and Ready.
7. Open the Showroom terminal. Run `oc get nodes` and `oc get storagecluster -A` to confirm no StorageCluster exists yet — Portworx is not installed.

### Key Takeaways

- KDS is the right choice for lightweight OCP Virt VM storage; Portworx Enterprise is the right choice when customers need enterprise data services, heterogeneous back-ends, or scale beyond what KDS covers
- The VMware migration wave is a real, quantified opportunity — field engineers can reference specific production numbers in customer conversations
- The lab cluster has no Portworx installed at this point; Module 2 begins the installation from scratch

### Infrastructure Notes

- No hands-on Portworx steps in this module; cluster access is verified but not exercised
- The `oc get storagecluster -A` command should return "No resources found" — this is expected and confirms the lab precondition
