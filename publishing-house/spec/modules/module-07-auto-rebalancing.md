# Module 07 — Discussion: Auto-Rebalancing and Multiple Back-ends

<!-- This is a facilitated discussion module. There are no hands-on steps. -->
<!-- Estimated duration: 15 minutes. Facilitator-led, no terminal required. -->

### Brief Overview

This module closes the lab with a facilitated discussion rather than hands-on exercises. Having completed Modules 1-6, participants have direct experience with Portworx installation, pool configuration, VM workloads, failover, and manual pool relocate. This discussion connects those experiences to production operational patterns: how Portworx auto-rebalances data automatically across heterogeneous back-ends, when to trigger manual vs. automatic rebalancing, and the real-world considerations that make Portworx Enterprise the right answer for large-scale VM-on-Kubernetes deployments. The goal is to give field engineers the talking points they need for the post-lab customer conversation.

### Audience and Time

- **Audience:** Red Hat Solutions Engineers, Architects, and Account Executives; Modules 01-06 completion recommended for maximum context
- **Estimated duration:** 15 minutes — facilitated group discussion, no hands-on steps
- **Format:** Facilitator-led discussion. No terminal, no OCP console required. Participants may refer back to their running cluster to illustrate points if time allows.

### Learning Objectives

- Describe how Portworx auto-rebalances data across heterogeneous back-end types (SSD, HDD, NVMe) without operator intervention
- Identify the conditions under which manual rebalancing is preferable to auto-rebalance in a production environment
- Apply best practices for mixing storage tiers in a production Portworx cluster to a real customer scenario

### Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | How auto-rebalance works (concept + policy triggers) | 4 min |
| 2 | Best practices for heterogeneous back-ends | 4 min |
| 3 | Manual vs. auto: when to use each | 4 min |
| 4 | Real-world production considerations and Q&A | 3 min |

### Detailed Steps

<!-- Discussion talking points for the facilitator. No CLI commands. -->

1. **Auto-rebalance overview:** Ask participants what they observed in Module 06 (manual relocate). Transition to the question: what if Portworx did that automatically? Explain that PX-AutoPilot (an enterprise feature) can trigger pool rebalancing based on policy rules — e.g., when any pool reaches 70% capacity, automatically migrate the largest volumes to less-utilized pools.

2. **Heterogeneous back-ends:** Describe the production scenario: a cluster with NVMe (hot tier), SSD (warm tier), and HDD (cold tier) block devices. Each tier maps to a separate Portworx pool and a separate StorageClass. PX-AutoPilot can observe volume I/O patterns and migrate volumes between tiers automatically — hot data stays on NVMe, cold data moves to HDD.

3. **Mixing SSD, HDD, and NVMe — best practices:**
   - Always assign dedicated pools per device class; never mix device types in a single pool
   - Use `io_profile: db` for NVMe pools, `io_profile: sequential` for HDD pools
   - Set replication to 2 minimum across tiers; cross-tier replication is supported
   - Size HDD pools 3-5x NVMe pools to accommodate data gravity over time

4. **Manual vs. auto-rebalance:**
   - Use auto-rebalance (PX-AutoPilot) for routine capacity management and tier migration
   - Use manual pool relocate (Module 06) when: decommissioning a node, responding to a hardware failure, or needing to force a specific data placement for compliance reasons
   - Auto-rebalance is eventually consistent; manual relocate is synchronous and verifiable

5. **Real-world production considerations:**
   - In a 45+ enterprise reference deployment, auto-rebalance eliminated 80% of manual storage interventions
   - Cross-AZ replication requires pool-level AZ awareness; Portworx supports cloud drive topology labels
   - Storage quota enforcement per namespace is a PX-Security feature that works alongside auto-rebalance
   - Open Q&A: what questions came up during the lab that map to a customer scenario you are currently working?

### Key Takeaways

- Auto-rebalance handles routine capacity management without operator intervention; manual relocate handles explicit, compliance-driven, or decommission scenarios
- Heterogeneous back-end clusters require per-tier StorageClasses; PX-AutoPilot provides the policy engine that moves data between tiers based on I/O patterns and capacity thresholds
- The combination of auto-rebalance and manual relocate gives Portworx operators both reactive and proactive data placement control
- Field engineers should lead with the vSAN replacement narrative and use the auto-rebalance story to differentiate from simpler CSI-only storage solutions

### Infrastructure Notes

- No cluster access required for this module; it is a facilitated discussion
- Facilitator may optionally run `pxctl status` on the lab cluster to show a live view of pool utilization as a discussion anchor
- PX-AutoPilot is an enterprise feature included in the Portworx Enterprise license; it is not demonstrated hands-on in this lab due to time constraints, but the concept is discussed here
