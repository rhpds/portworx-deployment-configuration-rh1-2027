# LB8002 full student E2E

Tracking: **RHDPCD-2096**. Ansible only: no custom Python scripts.

## Contract and layout

`healthcheck.yml` checks a fresh CI before the student starts.
`e2e.yml` now **performs the student exercises**, then validates their outcomes;
it no longer requires a manually solved environment. `e2e.yaml` is an alias.
Source of truth: `content/modules/ROOT/pages/module-01.adoc` through `module-06.adoc`.

The entry point imports these playbooks sequentially:

| Playbook | Exercise |
| --- | --- |
| `modules/module-01.yml` | Create the student project, enable monitoring, install Portworx operator/plugin and StorageCluster, wait for three online nodes, deploy HyperConverged, wait for storage classes and CentOS boot image, download matching virtctl, validate installation |
| `modules/module-02.yml` | Generate an exercise SSH key, create authorized-keys, provision two CentOS VMs, wait for guest access and validate RWX Block disks |
| `modules/module-03.yml` | Write guest data, initiate a new cross-node live migration, verify persistence, reboot the hosting worker, require VM failover/restart on another node, verify data and node/storage recovery |
| `modules/module-04.yml` | Take a new snapshot, delete and confirm missing guest /etc/fstab, stop VM, restore this snapshot, restart VM and compare the restored checksum |
| `modules/module-05.yml` | Provision the Autopilot VM with a 5Gi disk, create the rule/labels, partition/format/mount, write 4Gi, wait for genuine automatic expansion and a new action event, grow the guest filesystem |
| `modules/module-06.yml` | Create the student StorageClass/PVC/fio Deployment, write 6Gi, measure random-read IOPS at 10/20/40Gi, verify 500/1000/1500 configured limits, opt out, measure unthrottled IOPS, verify no workload restart, clean up |

Shared settings are in `vars.yml`. Student execution is in
`tasks/student-module-*.yml`; existing `tasks/module-01.yml` through
`module-05.yml` supply validation checkpoints. The older isolated
`tasks/module-06.yml` smoke fixture is not invoked by the full workflow.

Console actions are reproduced through Kubernetes APIs. This provides equivalent
resource operations, **not browser/UI coverage**. Observed fio throughput is
recorded, not required to reach each ceiling: the lab disks can be the bottleneck.

## Run

Use a **disposable, dedicated LB8002 cluster** in the fresh health-check state.
The OpenShift Virtualization operator is preinstalled; HyperConverged and
Portworx are installed by module 01. No manual solver is required.

The runner needs `ansible-core`, `kubernetes.core`, its Kubernetes client
dependencies, `oc`, `ssh`, `ssh-keygen`, gzip/tar support, cluster-admin access,
and access to the catalog, workload images, boot images and client downloads.
The generated exercise SSH key stays in `e2e_work_dir`, outside the repository
and separate from exported result JSON. Never commit or publish private keys.

```sh
export KUBECONFIG=/path/to/lab.kubeconfig
ansible-playbook -i localhost, qa-automation/healthcheck.yml

# Full student workflow: includes worker reboot and guest snapshot/restore.
ansible-playbook -i localhost, qa-automation/e2e.yml

# A standalone module (earlier module prerequisites must already exist):
ansible-playbook -i localhost, qa-automation/modules/module-02.yml

# Or select a module through the main entry point:
ansible-playbook -i localhost, qa-automation/e2e.yml --tags module04
```

`e2e.yaml` accepts the same options. `K8S_AUTH_KUBECONFIG` takes precedence over
`KUBECONFIG`; `-e k8s_kubeconfig=...` overrides both. Polling defaults to 90
retries with a 10-second delay. Override shared settings with `-e`.

Starting E2E through the Showroom curl endpoint runs all six student modules
without an enable flag or extra environment configuration. Module 03 performs
the worker reboot and module 04 performs the guest snapshot/restore exercise.
The obsolete `e2e_allow_disruptive` and `LB8002_E2E_ALLOW_DISRUPTIVE` flags are
not consulted. Resource-ownership and safe reboot-target checks remain mandatory.
Only invoke this workflow on a disposable, dedicated QA cluster.

### Self-contained HA preparation and public Showroom streaming

No catalog settings or manual Showroom placement patch are required:

```sh
curl --http1.1 --fail-with-body --no-buffer "$SHOWROOM_URL/stream/qa/e2e"
```

Module 03 first uses a worker that already hosts neither Showroom nor ingress.
If Showroom and two ingress routers occupy all three workers, E2E prepares its
own HA target without moving/restarting Showroom:

1. Identify this runner pod and the current E2E request peer in its access log.
2. Map that peer to a healthy ingress router, protecting the router/node carrying
   the curl stream. Older E2E requests and health checks are not used as the current peer.
3. Choose another Ready, schedulable worker outside Showroom and the active stream path.
4. Temporarily cordon that worker and evict only its named default-router pods,
   using namespace-scoped, UID-guarded `policy/v1` eviction requests that respect PDBs.
5. Wait for the ingress controller to restore router redundancy away from that worker.
6. In an `always` block, uncordon only the same worker owned by this invocation and
   remove the temporary marker, including when preparation fails.
7. Re-read actual pod placement before migrating the VM and performing the real reboot.

No Showroom/ingress Deployment, affinity policy, catalog, or provisioning setting
is rewritten. Replacement router pods remain where the normal scheduler places
them. Other clients using an evicted router may need to reconnect; this is a
disruptive test on a disposable dedicated cluster, not production maintenance.

Automatic preparation requires two healthy default routers on distinct nodes.
For streamed execution, the runner access-log peer must map uniquely to a healthy
router pod/host IP. If identification or redundancy checks fail, E2E refuses to
guess or evict a potentially active stream router. External CLI execution does
not require an SSE access log. Preexisting cordons and other runs' ownership
markers are never adopted or cleared.

Long waits use retry/async progress output to keep the SSE connection active.
A curl/network error is not a full streamed-test pass, even if module JSON later
shows the underlying exercises succeeded.

## Load testing and results

Run **one workflow per disposable cluster**, scaling load across separately
ordered CIs. This installs cluster-scoped operators, storage, monitoring and
Autopilot rules and reboots a worker; parallel workflows on a shared cluster
are not supported. A load-test scheduler/concurrency controller is not included.

Each module writes `module-01.json` through `module-06.json` under
`e2e_artifacts_dir` (default `/tmp/lb8002-e2e-results`) with status, elapsed
seconds and the failed task name. The first failure stops subsequent modules.
Module 06 also saves `fio-10Gi.json`, `fio-20Gi.json`, `fio-40Gi.json`, and
`fio-unthrottled.json` with the actual fio results, sampled after each limit
transition. Export these files before destroying the CI or restarting Showroom:
the default directories are ephemeral local runner storage.

The main entry point clears previous module/fio result JSON before starting.
Module 02 resets only the E2E-owned student namespace, recreating VMs, disks and
snapshot evidence so a full workflow can run again on the same dedicated CI.
Module 06 likewise resets its owned workload namespace before creating the PVC.
Module 05 requires an unexpanded 5Gi disk: rerun the full pipeline (or module 02
first), not module 05 alone against a previously expanded VM. Use a new
`e2e_artifacts_dir` per run if prior results must be retained.

## Disruption, ownership and cleanup

Namespaces `vmtest` and `appio` must be absent or carry the ownership label
`lab.rhpds.com/e2e=lb8002`. E2E refuses to adopt existing student namespaces or
an unowned dynamic-I/O StorageClass. Do not relabel student resources to bypass
this protection. Operator/monitoring changes remain cluster-wide.

Module 03 automatically prepares a router-free worker if necessary, and rechecks
that no Showroom or ingress pod is present before rebooting it. If the VM already
occupies the only safe worker, it first migrates away, then performs the measured
migration back onto that worker.
For another in-cluster runner, set `-e e2e_runner_node=<node-name>` (or set
`NODE_NAME` in its environment). If no safe worker is available, the module
fails rather than rebooting the runner. A brief reboot that does not actually
produce cross-node VM failover is a failed HA test, not a claimed success.

Modules 01-05 leave installation, VMs, snapshots and the rule for inspection.
A subsequent module 02 run deletes/recreates the owned exercise namespace;
never run it while inspecting or using the preceding run's VMs. Module 06 cleans up its owned workload namespace/StorageClass
in an `always` block, including on failure, and reports cleanup failures.
Destroy the disposable CI after collecting results. This is not a teardown of
student resources or an unattended production test.

## Offline validation

```sh
ansible-playbook -i localhost, qa-automation/e2e.yml --syntax-check
for playbook in qa-automation/modules/*.yml; do
  ansible-playbook -i localhost, "$playbook" --syntax-check
done
ansible-playbook -i localhost, qa-automation/tests/virtctl-selection.yml
ansible-playbook -i localhost, qa-automation/tests/student-workflow.yml
ansible-playbook -i localhost, qa-automation/tests/ha-topology.yml
yamllint -d '{extends: default, rules: {line-length: {max: 120}}}' \
  qa-automation/e2e.yml qa-automation/e2e.yaml qa-automation/vars.yml \
  qa-automation/modules/ qa-automation/tasks/ qa-automation/tests/
```

Syntax, fixture and lint tests are not a live six-module acceptance run. Only
report a full E2E pass after all six exercises succeed on a fresh lab.
