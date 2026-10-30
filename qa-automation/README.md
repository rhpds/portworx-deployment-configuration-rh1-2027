# LB8002 QA automation

Tracking: **RHDPCD-2096**. Ansible only: no custom Python scripts.

## Two different contracts

- `healthcheck.yml`: fresh-environment readiness before the student starts.
- `e2e.yml`: **post-solve module validation**, based on
  `content/modules/ROOT/pages/pxe-osv-01.adoc` through `pxe-osv-06.adoc`.
  It does not implement a solver or replay every student step.

Prepare module outcomes with a solver or a completed QA walkthrough first.
The fresh health check must not be included in post-solve E2E: fresh checks
expect Portworx and HyperConverged not to be deployed yet.

## Per-module validation

| Task file | Validates |
| --- | --- |
| `tasks/module-01.yml` | Ready cluster, three online Portworx storage nodes, Available HyperConverged, Portworx CSI classes, default VM class, RWX Block profile, Ready CentOS boot source, enabled console plugin, available STORK/Autopilot and user workload monitoring |
| `tasks/module-02.yml` | Ready VM, Running VMI, Bound Portworx RWX Block disk and working guest SSH |
| `tasks/module-03.yml` | Successful cross-node migration record and intact guest persistence marker. Module 01 checks recovered node/storage health |
| `tasks/module-04.yml` | Ready snapshot, completed restore referencing that snapshot, nonempty guest `/etc/fstab` |
| `tasks/module-05.yml` | Rule selectors/threshold/action, selected namespace/PVC, 10Gi or 20Gi expanded capacity, completed Autopilot action event, guest filesystem grown beyond original 5Gi |
| `tasks/module-06.yml` | Isolated 10 → 20 → 40Gi PVC smoke test, enforced read/write limits of 500 → 1000 → 1500 IOPS, persistent data across expansion and opt-out retaining manually disabled limits |

`tasks/pxctl-target.yml` avoids stale pods on unavailable nodes.
`tasks/module-06-resize.yml` performs each smoke expansion/assertion.
Missing module evidence fails validation; it is not silently skipped.

This is outcome validation, not a claim that every student action was tested:
no node failure is injected or proven, no destructive guest file deletion or
snapshot restore is repeated, and no browser-rendering check is performed.
The dynamic I/O smoke checks configured/enforced limits using `pxctl`, not fio
throughput, which the content notes may be limited by the underlying disks.

## Solver prerequisites

Retain these artifacts on a disposable, solved LB8002 environment:

- Installation/configuration from module 01.
- Namespace `vmtest`, VM `centos-stream9-example`, and its authorized SSH key.
- Guest `~/text` containing `Hello from Red Hat and Portworx`.
- A successful cross-node migration record for that VM.
- A Ready VM snapshot and completed restore from it; VM running again with a
  nonempty `/etc/fstab`.
- VM `centos-stream9-autopilot`, PVC `centos-stream9-autopilot-data-disk`, rule
  `volume-resize`, and the namespace/PVC `expand=true` labels. Autopilot has
  grown the 5Gi disk and the solver has grown the guest partition/filesystem.
- A recent `ActiveActionsTaken` event for `volume-resize`.

Migration records and Kubernetes events are ephemeral. Run E2E promptly after
solving. If evidence has expired, rerun that solver exercise rather than ignore
it. Events establish that the rule acted, not exclusive proof that this PVC's
size was changed by Autopilot rather than by an administrator.

Module 06 cleans up student resources in the content, so validation does not
require `appio` or `px-dynamic-io` to exist. Instead it provisions a lightweight
smoke fixture with the same StorageClass parameters and image as the content.
It writes a tiny marker, not a multi-GiB fio dataset.

## Run

Run from an execution environment / bastion with `ansible-core`,
`kubernetes.core`, its Kubernetes client dependencies, `oc`, `virtctl`, cluster
admin access, and the private key authorized by the solver VM.

```sh
export KUBECONFIG=/path/to/lab.kubeconfig
ansible-playbook qa-automation/e2e.yml \
  -e e2e_ssh_key=/path/to/solver-guest-key

# Validate only an individual module:
ansible-playbook qa-automation/e2e.yml --tags module04 \
  -e e2e_ssh_key=/path/to/solver-guest-key
```

`K8S_AUTH_KUBECONFIG` takes precedence over `KUBECONFIG`.
Polling defaults to 90 retries with a 10-second delay (`e2e_retries`,
`e2e_delay`). HCO/consumer readiness waits are capped at 900 seconds and guest
SSH tasks at 60 seconds per attempt. Opt-out verification waits 60 seconds
(`e2e_autopilot_settle_seconds`, minimum 30); increase this if Autopilot uses a
longer reconciliation interval.

To avoid all smoke writes, pass `-e e2e_module06_enabled=false`. Module 06 is
then **not validated**: this must not be reported as a full six-module pass.

## Safety and cleanup

Modules 01–05 read API / guest evidence only. No worker reboot, VM restart,
snapshot restore or solver-resource modification occurs.

Module 06 creates a generated `lb8002-e2e-*` namespace and a matching
`px-lb8002-e2e-*` StorageClass with Delete reclaim policy. An `always` block
attempts both cleanups and reports any cleanup failure. It allocates up to 40Gi
logical replicated storage: use a QA lab with sufficient free capacity only.
Deleting the namespace requests PVC/PV/backend-volume cleanup; backend reclaim
may finish asynchronously. After interruption or cleanup failure, inspect these
resources and their PVs. Never delete similarly named student resources.

Guest SSH disables host-key checking only for ephemeral lab guests. Never run
with production cluster credentials.

## Static validation

```sh
ansible-playbook qa-automation/e2e.yml --syntax-check
ansible-lint qa-automation/e2e.yml
yamllint -d '{extends: default, rules: {line-length: {max: 120}}}' \
  qa-automation/e2e.yml qa-automation/tasks/
```

Syntax/lint checks do not replace a solved-lab execution. No live lab result is
implied by static validation.
