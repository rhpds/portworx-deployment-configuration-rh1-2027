# portworx-deployment-configuration-rh1-2027

 Migration of the existing Portworx Enterprise lab (pxe-rhdp-lab) into the Publishing House workflow for Red Hat One 2027. This lab covers Portworx Enterprise deployment and configuration on
  OpenShift as a vSAN/datastore replacement for OCP Virtualization environments. Scope: Kube Datastore overview (KDS vs PXE), installing Portworx Enterprise via the OCP Operator catalog,
  managing storage and dynamic storage pools, deploying VM workloads on RWX block storage, simulating node failover and storage recovery, manual pool relocation, and auto-rebalancing across
  heterogeneous backends. Target audience: Red Hat field (SEs, architects, sellers). Duration: approximately 2 hours.

**Owner:** rickgcv
**Migrated from:** https://github.com/PureStorage-OpenConnect/pxe-rhdp-lab

---

## What was set up

1. Repository created (migrated from existing Showroom repo)
2. `catalog-info.yaml` added to repository
3. Registered in Developer Hub catalog
4. Orchestrator workflow started — your AI-guided content pipeline is running!

## What happens next

Claude will walk you through the entire content lifecycle — from intake and spec creation, through Jira tracking and reviews, all the way to a published lab on RHDP. Just follow the prompts!

## Getting started

### DevSpaces (recommended)

1. Open in DevSpaces: `https://devspaces.apps.ocpv-infra02.wdc07.infra.demo.redhat.com#https://github.com/rhpds/portworx-deployment-configuration-rh1-2027`
2. Use Claude via the **extension** or the **CLI**:
   - **Extension:** Click the **Claude** icon in the sidebar, click **New Session**. If the Claude icon is not visible, open **Extensions** (`Ctrl/Cmd+Shift+X`), find **Claude Code for VS Code** under the DevSpaces section, click it, then click **Enable (Workspace)**.
   - **CLI:** Open a terminal and run `claude`
3. Run `/rhdp-publishing-house` — and you're off!

### Local machine

1. Install the skills:
   ```
   git clone -b prod https://github.com/rhpds/rhdp-publishing-house-skills.git ~/.claude/skills/publishing-house
   ```
2. Clone the repo:
   ```
   git clone https://github.com/rhpds/portworx-deployment-configuration-rh1-2027
   ```
3. `cd portworx-deployment-configuration-rh1-2027`
4. Start Claude CLI: `claude`
5. Run `/rhdp-publishing-house` — and you're off!
