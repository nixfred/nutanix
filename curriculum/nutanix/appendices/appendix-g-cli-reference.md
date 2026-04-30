---
appendix: G
title: CLI Reference
type: reference
purpose: |
  Command syntax and usage patterns for the Nutanix CLI surface. Covers ncli
  (cluster management), acli (Acropolis VM/storage operations), NCC (health
  checks), the OVS commands AHV uses for networking, manage_ovs, and the Move
  CLI. Includes diagnostic recipes that combine commands to answer common
  troubleshooting questions.
usage: |
  Read on demand during operations work, troubleshooting, or POC validation.
  Most CLI work for ongoing operations should go through Prism. CLI shines
  during break-fix, customer demos that need verification, and scripting.
discipline: |
  Three principles:
  1. Commands evolve. Always check `command --help` before relying on syntax
     from any document, including this one.
  2. Run dry-runs and read-only commands first. Mutating commands can affect
     production.
  3. Document what you ran. Customer environments deserve change records.
last_updated: 2026-04-30
covers:
  - SSH access patterns (CVM and host)
  - ncli (cluster management, common commands)
  - acli (Acropolis operations, VM and storage)
  - NCC (health checks)
  - OVS commands (Open vSwitch on AHV)
  - manage_ovs (Nutanix OVS management wrapper)
  - Move CLI (migration tooling)
  - Diagnostic recipes (compound CLI patterns)
---

# Appendix G: CLI Reference

The Nutanix CLI surface is large. This appendix covers the commands BlueAlly SAs reach for most often: cluster diagnostics, VM operations during POCs, storage validation, networking verification, and Move troubleshooting. It is a working reference, not a complete man page; for full options always run `command --help` against the version you're working with.

**Important caveat:** command syntax evolves between AOS versions. Verify against the version running in the environment. If you find this document is out of date for a specific command, defer to `--help` and the official docs at portal.nutanix.com.

---

## SSH Access

### Connecting to a CVM

```bash
ssh nutanix@<cvm-ip>
```

Default user is `nutanix`. Authentication via SSH key or password depending on cluster configuration. Once on a CVM, you can run cluster-wide commands like `ncli`, `acli`, and `cluster status`.

### Connecting to an AHV Host

```bash
ssh root@<ahv-host-ip>
```

Default user on AHV hosts is `root`. SSH key auth typical. Use this for host-level networking work (OVS, bonds), or when troubleshooting AHV itself.

### Running Commands Across All CVMs

From any CVM, use `allssh` to run a command on every CVM in the cluster:

```bash
allssh "ncli cluster info"
```

For host-level commands across all AHV hosts:

```bash
hostssh "ovs-vsctl show"
```

These wrappers respect cluster-level connectivity and are the standard pattern for fleet-wide queries.

---

## ncli (Cluster Management)

`ncli` is the primary CLI for cluster-level management. Operations include cluster info, host management, storage container management, network configuration, replication, and licensing.

### Common Patterns

**Get cluster info:**

```bash
ncli cluster info
```

Returns cluster name, ID, version, member CVM IPs, configured storage pool, redundancy factor, and similar high-level state.

**List storage containers:**

```bash
ncli storage-container list
```

Output shows each container's name, ID, RF, compression state, dedup state, and capacity.

**Show storage utilization:**

```bash
ncli cluster get-domain-fault-tolerance-status
```

Reports cluster-wide fault tolerance and capacity headroom.

**List hosts:**

```bash
ncli host list
```

Returns host IDs, names, hypervisor, status, CPU and memory totals.

**Run a health check:**

```bash
ncli health-check run-all
```

Equivalent to NCC's full check; can take several minutes on large clusters.

**Check replication state:**

```bash
ncli protection-domain list
ncli protection-domain list-snapshots
```

For the legacy Protection Domain construct. For Protection Policies (the modern construct), use Prism Central or the v4 API.

**Add or list licenses:**

```bash
ncli license list
ncli license add file=<license-file>
```

Used during license activation.

### Key ncli Object Types

| Object | Purpose |
|---|---|
| `cluster` | Overall cluster state |
| `host` | Individual nodes |
| `storage-container` | Datastore-equivalent |
| `storage-pool` | Aggregate of physical disks |
| `network` | Virtual networks |
| `protection-domain` | Legacy DR grouping |
| `data-services-vip` | Volumes-related IP |
| `vm` | Virtual machines (operations) |
| `health-check` | NCC integration |

Run `ncli help` for the full list.

### ncli Tips

- Most subcommands accept `--help` for detailed syntax.
- Output is structured; pipe to `grep`, `awk`, or `jq` (after JSON conversion) for scripting.
- Some commands require `nutanix` user and run on any CVM; others require explicit cluster-wide context.

---

## acli (Acropolis Operations)

`acli` is the AHV-specific CLI for VM lifecycle, storage operations on AHV containers, and network management. It runs on the CVM.

### VM Operations

**List VMs:**

```bash
acli vm.list
```

**Get VM detail:**

```bash
acli vm.get <vm-name>
```

Shows configuration, attached disks, NICs, host placement, snapshots.

**Create a VM:**

```bash
acli vm.create <vm-name> num_vcpus=4 num_cores_per_vcpu=1 memory=8G
acli vm.disk_create <vm-name> create_size=100G container=<container-name>
acli vm.nic_create <vm-name> network=<network-name>
acli vm.on <vm-name>
```

**Stop a VM:**

```bash
acli vm.shutdown <vm-name>     # graceful
acli vm.off <vm-name>          # forced
```

**Migrate (live):**

```bash
acli vm.migrate <vm-name> host=<target-host>
```

**Snapshot a VM:**

```bash
acli vm.snapshot_create <vm-name> snapshot_name=<name>
```

**Clone a VM:**

```bash
acli vm.clone <new-vm-name> clone_from_vm=<source-vm-name>
```

### Storage Operations

**List containers:**

```bash
acli storage_container.list
```

**Show container detail:**

```bash
acli storage_container.get <container-name>
```

**Create a container:**

```bash
acli storage_container.create <name> rf=2 compression_type=on dedup=off
```

### Network Operations

**List virtual networks:**

```bash
acli net.list
```

**Get network detail:**

```bash
acli net.get <network-name>
```

**Create a virtual network:**

```bash
acli net.create <network-name> vlan=<vlan-id>
```

For IPAM-managed network:

```bash
acli net.create <network-name> vlan=<vlan-id> ip_config=<network>/<prefix>
```

**Add a DHCP range:**

```bash
acli net.add_dhcp_pool <network-name> start=<ip> end=<ip>
```

### acli Tips

- Tab completion works for object names; use it.
- `acli` parses dotted-method syntax: `vm.get`, `storage_container.list`, `net.create`.
- For programmatic work, the v4 REST API is usually a better choice than parsing acli output.

---

## NCC (Nutanix Cluster Check)

NCC is the cluster's built-in health-check framework. Hundreds of individual checks across hardware, software, configuration, and performance.

### Common Patterns

**Run all health checks:**

```bash
ncc health_checks run_all
```

This is the first command to run when something is wrong. Output shows PASS, INFO, WARN, FAIL per check, with explanations and recommendations.

**Run a specific check:**

```bash
ncc health_checks <category>_checks <specific_check>
```

Example:

```bash
ncc health_checks hardware_checks disk_status_check
```

**List available checks:**

```bash
ncc health_checks list
```

**Get cluster log dumps for support:**

```bash
ncc log_collector run_all
```

Creates a tarball of logs to attach to support cases.

### NCC Categories

- `hardware_checks` (disk, NIC, memory, BMC)
- `network_checks` (connectivity, VLAN, OVS state)
- `cluster_checks` (cluster-wide consistency)
- `data_protection_checks` (replication, snapshots)
- `system_checks` (service health, version)
- `metadata_checks` (Cassandra, Stargate)

For the full list, `ncc health_checks list` is the source of truth on the running version.

---

## OVS Commands (Open vSwitch on AHV)

AHV uses Open vSwitch as the kernel-level virtual switch. SSH to the AHV host (not the CVM) to run OVS commands.

### Inspecting Bridges and Ports

**List bridges:**

```bash
ovs-vsctl show
```

Output shows `br0` (data bridge), `br0.local` (management bridge), and the ports on each.

**List bridges concisely:**

```bash
ovs-vsctl list-br
```

**List ports on a bridge:**

```bash
ovs-vsctl list-ports br0
```

**Show interface details:**

```bash
ovs-vsctl list interface eth0
```

### Bond Status

**Show bond state:**

```bash
ovs-appctl bond/show <bond-name>
```

Reports bond mode, member status (active/standby), LACP state if applicable.

**Show LACP detail:**

```bash
ovs-appctl lacp/show <bond-name>
```

### Flow Rules

**Dump flow rules on a bridge:**

```bash
ovs-ofctl dump-flows br0
```

Shows the OpenFlow rules currently programmed; useful for verifying Flow Network Security policy enforcement.

### OVS Caveats

- Direct OVS modification is not the typical path on AHV; use `manage_ovs` (next section) for Nutanix-aware operations.
- OVS commands are read-mostly during normal operations. Modification typically only during troubleshooting under support guidance.

---

## manage_ovs (Nutanix OVS Wrapper)

`manage_ovs` is the Nutanix-aware wrapper around OVS for bond and uplink management. Run on the AHV host.

### Common Patterns

**Show current bond config:**

```bash
manage_ovs --bridge_name br0 show_uplinks
```

**Configure a bond mode:**

```bash
manage_ovs --bridge_name br0 --bond_name br0-up --bond_mode active-backup update_uplinks
```

Bond modes:
- `active-backup`
- `balance-slb`
- `balance-tcp` (LACP, requires switch coordination)

**Add or remove uplinks:**

Refer to current Nutanix documentation for the precise syntax for adding/removing physical NICs from a bond. The flags evolve between AOS versions.

### manage_ovs Tips

- Run `manage_ovs --help` on the target host for current syntax.
- Bond changes affect networking on that host; coordinate with the cluster (live migrations off, etc.) before making changes during production hours.
- Always verify post-change state with `ovs-appctl bond/show`.

---

## Move CLI

Move primarily runs as a VM with a web UI. The CLI is for programmatic use, troubleshooting, and bulk operations.

### Common Patterns

**SSH to the Move VM:**

```bash
ssh admin@<move-vm-ip>
```

**Show Move version and status:**

```bash
move version
move status
```

**List configured environments (sources/targets):**

```bash
move env list
```

**List migration plans:**

```bash
move plan list
```

**Show plan detail:**

```bash
move plan show <plan-id>
```

**Trigger plan operations:**

```bash
move plan start <plan-id>
move plan cutover <plan-id>
move plan cancel <plan-id>
```

### Move CLI Tips

- The web UI is the standard interface; CLI is for automation and edge cases.
- Move log files (under `/opt/xtract-vm/logs/` typically) are the source for migration troubleshooting.
- For large-scale migrations, scripting against the Move REST API is more typical than CLI.

---

## Diagnostic Recipes

Compound CLI patterns for common troubleshooting questions.

### "Is the cluster healthy?"

```bash
# From any CVM
ncc health_checks run_all
ncli cluster info
cluster status
```

`cluster status` reports the state of every Nutanix service on every CVM: Stargate, Cassandra, Curator, Pithos, Zeus, Genesis, etc. All should be `UP`.

### "Why is storage performance bad?"

```bash
# CVM-level latency observations
ncli cluster get-domain-fault-tolerance-status
ncli storage-pool list

# Stargate stats
allssh "links http://0:2009/h/stargate"
```

The Stargate 2009 page exposes per-node I/O metrics; hot-spotted nodes show up here. The pattern is also visible in Prism Element and Prism Central performance views.

### "Which VMs are running on which host?"

```bash
acli vm.list
acli host.list
```

For VMs on a specific host:

```bash
acli host.list_vms <host-name>
```

### "What's the rebuild status after a failure?"

```bash
ncli cluster get-domain-fault-tolerance-status
ncli storage-pool list
```

The fault tolerance command reports current redundancy state; a node loss shows up as reduced fault tolerance with rebuild in progress.

### "Did the snapshot succeed?"

```bash
acli vm.snapshot_list <vm-name>
ncli protection-domain list-snapshots
```

For Protection Policies (modern construct), check via Prism Central or v4 API.

### "What's wrong with my OVS bond?"

```bash
# On the affected AHV host
ovs-vsctl show
ovs-appctl bond/show br0-up
manage_ovs --bridge_name br0 show_uplinks
```

Cross-check switch-side LACP state from the network team if using LACP modes.

### "Why is replication bandwidth saturated?"

```bash
# CVM-level
ncli protection-domain list
ncli protection-domain get-snapshot-progress

# WAN-level (from CVM, looking at outbound)
allssh "iftop -i eth0"   # if iftop available
```

Combine with monitoring at the customer's WAN edge for full visibility.

### "What version of AOS is running?"

```bash
ncli cluster info
# Look for "Cluster Version"

# More detail
upgrade_status
```

`upgrade_status` shows the LCM-managed component versions cluster-wide.

### "How do I generate a log bundle for support?"

```bash
ncc log_collector run_all
```

Output is a tarball under `/home/nutanix/data/log_collector/` typically. Upload to the support case as instructed.

---

## Common Mistakes with the CLI

1. **Running mutating commands without dry-run.** Many `acli` and `ncli` commands have no undo. Read the help; consider read-only verification first.
2. **Forgetting to `allssh` for cluster-wide queries.** Single-CVM queries miss cluster-wide state.
3. **SSH'ing to AHV hosts when CVM is the right target.** `ncli`, `acli`, and most cluster operations are CVM-resident. AHV hosts are for host-level networking and the hypervisor itself.
4. **Editing OVS directly when `manage_ovs` exists.** Direct OVS edits can break Nutanix's expected configuration.
5. **Treating CLI as the primary management surface.** Prism is the customer-facing path; CLI is for break-fix, scripting, and verification.
6. **Skipping `--help`.** Syntax evolves; the running version is the authority.

---

## Cross-References

- **Modules:** Each command set links to the module where the underlying concept is taught.
- **Glossary:** [Appendix A](./appendix-a-glossary.md) defines the terms used in commands.
- **POC Playbook:** [Appendix J](./appendix-j-poc-playbook.md) uses many of these commands for validation steps.
- **Reference Architectures:** [Appendix I](./appendix-i-reference-architectures.md) names the post-deployment validation commands.
- **Nutanix portal:** portal.nutanix.com has the authoritative current syntax for every command.
