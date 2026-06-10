# NCP-CN 6.10 — Full Objective Map (from the official blueprint)

Source: Nutanix Certified Professional - Cloud Native (NCP-CN) 6.10 Exam Blueprint Guide, Nutanix University. Local PDF: `./NCP-CN-6.10-Exam-Blueprint-Guide-official.pdf`. Tested versions: **NKP 2.12, AOS 6.10, Prism Central pc2024.2**. 75 questions / 120 min / $200 / pass 3000 (scale 1000-6000). Questions per objective are weighted by job-role criticality; Nutanix publishes no section percentages.

Intended audience: 6-12 months building/managing Kubernetes clusters, 6-12 months Linux, 6 months with a Kubernetes platform, **CKA-level knowledge assumed**. Recommended course: **Nutanix Kubernetes Platform Administration (NKPA)**.

---

## Section 1 — Prepare the Environment for an NKP Deployment

### Objective 1.1 — Seed a private registry
- Identify the purpose for seeding a registry
- Identify parameters for registry seeding command
- Demonstrate understanding of the workflow for seeding a registry
- Troubleshoot issues that arise during seeding the registry
- Recognize the network requirements for a private registry
- Refs: Air-gapped vs Non-Air-gapped Environments; Seeding the Registry for an Air-gapped Cluster; Prerequisites for Installation; Nutanix Air-gapped: Loading the Registry

### Objective 1.2 — Create a bootstrap cluster
- Identify the purpose for creating a bootstrap cluster
- Identify parameters for creating a bootstrap cluster
- Demonstrate understanding of the workflow for creating a bootstrap cluster
- Troubleshoot issues during bootstrap cluster creation
- Refs: CAPI Concepts and Terms; Nutanix Air-gapped Installation; NKP Prerequisites; Outputting the Bootstrap Cluster Kubeconfig to a File

### Objective 1.3 — Determine license tiers for clusters
- Differentiate between the feature levels per license type
- Align the use case with a license tier
- Identify the process for obtaining a license
- Refs: NKP Starter License; NKP Pro License; NKP Ultimate License; Add an NKP License; Remove an NKP License

### Objective 1.4 — Prepare a bastion host
- Identify the purpose of a bastion host
- Demonstrate understanding of the bastion host prerequisites and components
- Recognize the network requirements for a bastion host
- Refs: Creating a Bastion Host; Basic Installations by Infrastructure

### Objective 1.5 — Build machine images or prepare nodes
- Purpose and process for building machine images with the NIB CLI tool
- Purpose and process for preparing nodes with the KIB CLI tool
- Recognize prerequisites and minimum requirements for building machine images / preparing nodes
- Determine how to customize the image-build / node-prep process
- Changes required to build an image in an air-gapped environment
- Refs: Konvoy Image Builder; KIB for AKS; Creating an Air-gapped Package Bundle; Konvoy Image Builder CLI; Pre-provisioned Air-gapped: Configure Environment; vSphere FIPS: Creating a CAPI VM Template; vSphere FIPS: Creating the Management Cluster

### Objective 1.6 — Gather information for building a cluster on a target provider
- Identify parameters for building a cluster on a target provider: networking requirements, connectivity details, storage requirements
- Refs: AKS Installation Options; Pre-provisioned: Defining the Infrastructure; General NKP Resource Requirements; Supported Operating Systems

---

## Section 2 — Manage Building an NKP Cluster

### Objective 2.1 — Customize and deploy clusters
- Use NKP CLI to build and deploy a cluster
- Given a use case, determine when/how to employ the various Cluster API components for customization
- Determine steps to diagnose a cluster deployment issue (which CAPI resources to analyze)
- Determine when and how to use a custom manifest for deploying clusters (corresponding CAPI resources + configuration parameters)
- Analyze the NKP CLI parameters required for a specific cluster deployment use case
- Refs: Prerequisites for Installation; Installing NKP; Pro and Ultimate Cluster Minimum Requirements; Nutanix Air-gapped Installation; Nutanix Air-gapped Environment Creating a New Cluster; Project Applications; NKP Insights Guide; AWS Installation Options; Creating vSphere Node Pools; Creating a New AWS Air-gapped Cluster

### Objective 2.2 — Customize and deploy Kommander and apply appropriate licenses
- Use NKP CLI to build and deploy Kommander
- Determine steps to diagnose a Kommander deployment issue
- Determine when/how to use a custom manifest for deploying Kommander (platform application resources + configuration parameters)
- Analyze the NKP CLI parameters required for a specific Kommander deployment use case
- Refs: FIPS Support in NKP; Using KIB with vSphere; BaseOS Image Requirements; Installing Kommander in a Small Environment; Additional Kommander Configuration

---

## Section 3 — Perform Day 2 Operations

### Objective 3.1 — Configure authentication and authorization
- Configure identity provider and groups
- Differentiate between Kommander roles and cluster roles
- Configure custom roles and role bindings
- Use tokens to authenticate users
- Environment contexts and role inheritance
- Using RBAC authorization
- Refs: NKP Security; Enforcing Policies Using Gatekeeper; External LDAP Directory Configuration; Access Control

### Objective 3.2 — Configure logging
- Understand the logging stack; enable logging stack applications
- Configure and manage the logging stack application
- Manage logging in a multitenant environment
- Gather logs; integrate persisting data to Nutanix Unified Storage; scale the logging stack
- Refs: NKP Logging; Logging Operator; Logging Stack Operator; Customizing Logging Stack Applications; Multi-Tenant Logging; Configuring Loki to Use AWS S3 Storage in NKP

### Objective 3.3 — Configure cluster backup and recovery
- Recognize dependencies for cluster backup and recovery
- Determine target storage requirements for cluster backup
- Configure Velero (schedule backups, enable)
- Use Velero CLI to perform backups and restores
- Configure storage/volume snapshot classes for backup
- Diagnose and address backup issues; perform a cluster restore
- Refs: NKP License Support for Backup & Restore; Backup Operations; Velero Configuration; Velero Installation Using CLI

### Objective 3.4 — Conduct performance and health monitoring
- Understand the monitoring stack; configure/enable it
- Centralize monitoring in a multi-cluster environment
- Customize service- and system-level metrics; use custom dashboards
- Diagnose and address performance and health issues
- Configure monitoring alerts and endpoints; configure backend storage for a monitoring application
- Refs: Centralized Metrics; Cluster Metrics; Monitoring and Alerts; NKP Insights Guide; Nutanix Management Tools

### Objective 3.5 — Configure cluster autoscaling
- Recognize the purpose and use cases for enabling/configuring cluster autoscaling
- Demonstrate how to enable and configure cluster autoscaling
- Refs: Configuring Nutanix Cluster Autoscaler; Configuring AWS Cluster Autoscaler; Configuring vSphere Cluster Autoscaler

### Objective 3.6 — Conduct lifecycle management functions
- Upgrade clusters; given a scenario, determine when/how to update cluster configurations
- Manage node pools; manually scale clusters; use the NKP CLI to delete clusters
- Refs: Upgrade Prerequisites; Upgrade: For Air-gapped Environments Only; Upgrade NKP Ultimate; Upgrade NKP Pro; Upgrading Kubernetes Version on a Managed Cluster

---

## Section 4 — Conduct NKP Fleet Management

### Objective 4.1 — Configure workspaces
- Given a use case, determine workload cluster workspace assignment
- Purpose and use cases for a workspace; configure infrastructure provider
- Configure access control at workspace level; deploy application to a workspace
- Purpose and use case of Insights
- Refs: Creating a Workspace; Deleting a Workspace; Workspace Applications; Enabling an Application per Cluster; Generating a Dedicated Login URL per Tenant; Multi-Tenancy in NKP; NKP Insights Overview; Installing NKP Insights Ultimate License

### Objective 4.2 — Deploy workload clusters to a workspace
- Impact of target provider environments on workload cluster deployments
- Purpose/use cases for deploying workload clusters; given a scenario, determine how to deploy
- Troubleshoot the deployment of a workload cluster
- Refs: Workspaces; Workspace Applications; Creating a Managed Cluster on VCD Through the NKP UI; vSphere Creating Managed Clusters Using the NKP CLI; Infrastructure Providers; Managing Access; Creating Workspace Role Bindings

### Objective 4.3 — Attach clusters to a workspace
- Purpose and use case for attaching clusters; given a scenario, determine how to attach
- Recognize capabilities of attached clusters; troubleshoot cluster attachments
- Configure and update attached clusters
- Refs: Basic Requirements for Attaching Existing Clusters; Requirements for Attaching Existing AKS, EKS, and GKE Clusters; Attaching an Existing Kubernetes Cluster; EKS: Preparing the Cluster; Prerequisites for a Tunneled Attachment; Creating a Default StorageClass

### Objective 4.4 — Detach or delete clusters from a workspace
- Purpose and use case of decommissioning
- Use the Kommander dashboard to delete a cluster from the GUI
- Troubleshoot a cluster detachment/deletion issue
- Refs: Disconnecting or Deleting Clusters; Delete an NKP Cluster with One Command; Deleting EKS Cluster from the NKP UI

### Objective 4.5 — Configure projects
- Purpose and use cases for a project; given a use case, determine workload cluster project assignments
- Configure access control at the project level; deploy applications at the project level
- Configure CD deployment; add clusters to a project; federate resources
- Refs: Creating a Project Using the UI; Continuous Deployment; Continuous Delivery with GitOps; Managing Access to Projects; Project Quotas and Limit Ranges

### Objective 4.6 — Configure platform applications
- Recognize platform application dependencies
- Customize a platform application deployment from the Kommander dashboard
- Differentiate between global and cluster scope application configuration
- Troubleshoot platform application configurations
- Modify, disable, or enable platform applications via CLI
- Refs: Platform Applications; Platform Application Dependencies; Ultimate: Enabling an Application Using the UI; Deployment Scope; Deploy Platform Applications Using CLI; Kommander Installation Based on Your Environment

---

## Recommended training (official)
**Nutanix Kubernetes Platform Administration (NKPA)**, free online or paid instructor-led: NKP concepts/install (air-gapped + connected), licensing, lifecycle (workspaces, create/attach/scale clusters, projects, upgrades), access control (IdPs, groups, workspace/project roles, bindings), platform applications, backup/restore, logging stack, monitoring (metrics, cost, Grafana, alerts).

## Additional resources (official)
Nutanix Community Edition; Test Drive; Nutanix Community cert forum; additional cloud-native resources listed in the blueprint.
