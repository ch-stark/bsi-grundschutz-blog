# Using ACM-Policies for flexibly generating and implementing Compliance-Standards using the Example of BSI-Grundschutz

## Introduction and Scope

Organizations require the need for achieving a high level of compliance and expecting guidance and support to achieve this level as easy, flexible, fast and structured as possible.
In the following blog we describe how this can be achieved using the example of the German-Federal-Standard called  BSI-Grundschutz. This blog also considers that you need to ensure that the standard works not only for a single but for a fleet of OpenShift- or other Kubernetes-Clusters.that can dynamically change. The blog also highlights the different roles which are implementing the steps to achieve the highlighted goal.

Links: 
Policy-Generator-Blog, 
Policies-Whitepaper, 
Policy-Collection-Repository, 
Kyverno, 
Compliance-Operator, 
https://github.com/ch-stark/maptoseveralstandards
https://github.com/stolostron/policy-collection
https://docs.openshift.com/container-platform/4.10/security/compliance_operator/compliance-operator-understanding.html
https://kyverno.io/
[Kubernetes Policy Workgroup White Paper](https://github.com/kubernetes/sig-security/blob/main/sig-security-docs/papers/policy/CNCF_Kubernetes_Policy_Management_WhitePaper_v1.pdf)
https://cloud.redhat.com/blog/generating-governance-policies-using-kustomize-and-gitops


## Recap on ACM-Policies 

For better explaining the forthcoming concepts let’s do a quick recap on the basic concepts of Policies. 

### Relevant fields within a Policy

We have three fields to map a Policy to the requirements defined in the standard:

Standards (BSI-Grundschutz)
Controls
Categories

### Grouping Policies

For better visibility and maintency you can group policies into PolicySets. This concept will be available both as a UI-feature and for use in Gitops-Scenarios (starting with ACM 2.5). 

### Templatized-Policies

Also starting with ACM 2.5 we introduce a protect function to ensure the Policy can encrypt certain content so that it cannot be reviewed after an ETCD-backup. This new feature can be used as an Out-of-the-box Secret-Management Solution.
An example might look like this where you pull the Secret from a Namespace in the Hub to the Managed-Cluster(s) where the Policy is applied:

```
apiVersion: policy.open-cluster-management.io/v1
kind: ConfigurationPolicy
metadata:
  name: demo-template-function
  namespace: test
spec:
  namespaceSelector:
    exclude:
    - kube-*
    include:
    - default
  object-templates:
  - complianceType: musthave
    objectDefinition:
    ...
    spec:
      enabled:  |
        {{hub "(lookup "v1" "Secret" "default" "my-hub-secret").data.message | protect hub}}
```

### How to implement Mandatory and Optional Policies

When going through the Selection-Process which Policies should be applied and in which priority one could classify the input resources as mandatory which must be implemented and should/recommended to have policies.

Here the Severity-Field within a Policy could be used.

It could be for example:

```
severity: medium
```

Or

```
severity: high or critical
```
Note:
  https://cloud.redhat.com/blog/using-policyreports-to-view-and-alert-for-governance-violations

In order to alert on any violations produced by a policy, set the severity of the policy to critical and follow the steps previously outlined in in the alerting blog to set up alerting on your cluster.




For further reading on those concepts you might check or the following [blog](https://www.opensourcerers.org/2021/10/11/rhacm-and-policies-more-details/)
or the [Kubernetes Policy Workgroup White Paper](https://github.com/kubernetes/sig-security/blob/main/sig-security-docs/papers/policy/CNCF_Kubernetes_Policy_Management_WhitePaper_v1.pdf)


### Administrative Tasks/Preparation

As a start of the Process a RHACM/OpenShift/Kubernetes-Administrator is providing the necessary resources and necessary configuration-files which can be used as an input for generating the desired results. In the following we follow a git/gitops-based approach
based on the below examples 

https://github.com/ch-stark/maptoseveralstandards/tree/main/config/input


This initial provisioning of input resources can/should have redundancy as we will later select only some parts for our Policies

## Selection of Compliance-Standard and Technology-Selection

### Selection of Compliance-Standard

Once the input has been finalized a Compliance-Officer/Security-Architect is selecting the Compliance-Standard which fits best the Organization. Often this standard is coming from Industry or Governance requirements.

In this blog we use the following Documents as source for the standard and its implementation.:

bsi grundschutz openshift
bsi grundschutz container

Note: We can only support the Technical-Compliance Standard not on the Process level. By following the described approach you will not achieve full-compliance but we will significantly
support you on your journey regarding this goal and it will significantly speed up Audit-Readiness.

In the following we would like to review and standard together with some implementation advices.


### Introduction and Safeguards regarding the Component APP.4.4 Kubernetes

Kubernetes has established itself as the de facto standard for orchestrating containers in the public and private cloud. Kubernetes is also used for IoT and other applications. With K3S, for example, there is an edition that is intended for very small servers such as single-board computers. The so-called Cloud Native Stack, which consists of many different components, is also based on the standard established by Kubernetes.
The term container describes a technique in which a host system runs applications in parallel in separate environments (operating system level virtualization). In most cases, the monitoring, starting and stopping and further management of the container is carried out by management software, which thus takes over the so-called orchestration. Kubernetes combines one or more related containers in a pod. Since the focus of the module is on Kubernetes, only pods and not individual containers will be discussed below. The orchestration usually takes place in groups of jointly managed Kubernetes nodes in one or more so-called clusters.
In order to operate the orchestration of pods and to manage them, several products have been established that also make it possible to serve very large environments. At their core, however, they are all based on Kubernetes. A distinction must be made between the runtime, which runs the processes on the Kubernetes nodes, and the orchestration, which controls the runtimes on several Kubernetes nodes.
In addition to these two central components, the operation of Kubernetes consists in most cases of a specialized infrastructure, which includes e.g. registries, code versioning and storage, automation tools, administration servers, storage systems or virtual networks.

### Safeguards

The following are specific safeguards for the APP.4.4 Kubernetes building block requirements.
The measures required to implement the requirements in the APP 4.4 Kubernetes module are largely specific to the respective distribution. Here, reference is made to the OpenShift Container Platform variant from Red Hat as an example.
All measures (marked with M) are numbered in ascending order and correspond to the corresponding requirements (marked with A).

### Safeguards regarding Module APP.4.4 Kubernetes

APP.4.4.M1 Application separation planning (B)

Before going live, it MUST be planned how the applications running in the pods and their different test and production operating environments will be separated. Based on the protection requirements of the applications, the planning MUST determine which architecture of the namespaces, meta tags, clusters and networks adequately addresses the risks and whether virtualized servers and networks should also be used.
The planning MUST contain regulations on the separation of network, CPU and permanent storage. The separation SHOULD also take into account the network zone concept and the protection requirements and be coordinated with them.
Applications SHOULD each run in their own Kubernetes namespace that encompasses all programs in the application. Only applications with similar protection requirements and similar possible attack vectors SHOULD share a Kubernetes cluster.
The protective measure is of an organizational nature. OpenShift fully supports it through various technologies:

### Architecture

The very generic advice is Build at least 2 OpenShift clusters for non-productive and productive application workload. More ideally building individual OpenShift clusters for different network zones and application classes (a combination is possible, but it requires separate configurations for separation within OpenShift)

Platform configuration (in addition to the architectural aspects mentioned)

Provision of different storage classes for dedicated separation of storage ([STORAGECLASS])

Definition of different worker node types to separate applications

Use of NetworkPolicies, egress firewalls and other network-side options. Network separation in Kubernetes is described in Chapter 6 (Network Security) of the Red Hat OpenShift Security Guide.

Use of quotas and limits/limit ranges to restrict resources

Application-specific deployment configuration ([RBAC], [SCC], [NODESELECTORS])
Separation via namespaces, taking into account other previously mentioned aspects in the platform configuration ([LABELS], [NODESELECTORS])

Use of meta information on the namespaces to support organizational aspects ([LABELS])

Organizational
Definition of clear development and deployment guidelines for the use of the architecturally, configurationally and operationally prepared separations

`Red Hat Advanced Cluster Management for Kubernetes` is used for the central management of the OpenShift Container Platform cluster and helps to implement the organizational protective measures by automating the configuration.
`Red Hat Advanced Cluster Security for Kubernetes` allows policy-based compliance checking on the OpenShift Container Platform clusters.

### APP.4.4.M2 Planing auf Automation using CI/CD (B)

When automating the operation of applications on Kubernetes using CI/CD, it MUST ONLY be done after proper planning. The planning MUST cover the entire lifecycle from commissioning to decommissioning including development, testing, operation, monitoring and updates. The role and rights concept as well as the protection of Kubernetes secrets MUST be part of the planning.

The protective measure is of an organizational nature. OpenShift fully supports it. With the integrated CI/CD technologies [JENKINS], [PIPELINES] and [GITOPS], OpenShift already offers preconfigured solutions for automated CI/CD pipelines. Of course, other technologies such as Gitlab-CI, GitHub-Actions can also be integrated. The [RBAC] model must be configured to run each CI and CD step with minimal privilege.
The images are scanned for vulnerabilities using Quay and Red Hat Advanced Cluster Security for Kubernetes. Corresponding scans should already be carried out in the pipeline during the build process.
Kubernetes secrets must be secured as part of the overall security concept. Here it should be checked whether storage in the (encrypted) ETCD metadata store of the platform is sufficient or whether additional integration of vault components or "sealed secrets" for CD and GitOps mechanisms is required. For both approaches, OpenShift solutions are available in the community.


### APP.4.4.M3 Identity and permissions management in Kubernetes

Kubernetes and all other applications of the control plane MUST authenticate and authorize every action of a user or, in automated operation, corresponding software, regardless of whether the actions take place via a client, a web interface or via an appropriate interface (API). Administrative actions MUST NOT be done anonymously.
Each user MUST ONLY be given the rights that are absolutely necessary. Unrestricted permissions MUST be very restrictive.
Only a small circle of people SHOULD be authorized to define automation processes. Only selected administrators SHOULD be given the right to create or change shares for permanent storage (Persistent Volumes) in Kubernetes.


OpenShift provides the necessary basis for the implementation of the measure via integration with an identity provider and the integrated [RBAC] (Role-based Access Control) concept.
In the basic configuration, OpenShift allows the use of the web console and the API only for authenticated users. The preconfigured roles enable easy assignment of authorizations according to the principles of least privilege and need-to-know. User actions can be traced via the audit log.
The RBAC model distinguishes between cluster and namespace permissions. In the basic configuration, the automation can therefore be authorized on a fine-grained basis.
In the basic configuration, Persistent Storage can only be integrated by cluster administrators. In the case of dynamically provisioned storage, the corresponding provisioners have the necessary authorizations. These provisioners must be set up and configured by an admin. Storage requirements are controlled and restricted using the quota mechanisms ([QUOTAS]).
If applications do not offer a sufficient authentication method, API management systems such as Red Hat 3scale API Management can support these applications on the infrastructure side here.
Basic recommendations for implementation:
Integration of one (or more) identity providers in OpenShift for user authentication
Definition (organizational) and implementation (technical/configurative in OpenShift) of a group concept
Using the RBAC model shipped with OpenShift
restrictive assignment of new authorizations via individually defined roles, in particular cluster-wide roles
Assignment of permissions via roles only to groups to avoid mixing permissions to individual user IDs
Use of quota definitions on application-specific namespaces to control usage of dynamically provisioned storage
Definition of own application-specific roles to enforce the principle of least privilege.

### APP.4.4.M4 Separation of PODS (B)

OpenShift uses the Red Hat Enterprise Linux CoreOS, which is designed for container operation, for the nodes. Optionally, Red Hat Enterprise Linux can also be used. In both configurations, CRI-O is the container runtime. Specifically, enforce at the system level
CGroups
seccomp
SELinux in 'enforcing' mode
the separation of the pods. [SCC] can fine-grained extend the rights, which are completely separate in the basic configuration, if necessary. 

### APP.4.4.M5 Data backup in the cluster (B)

The cluster MUST be backed up. The backup MUST include:
permanent storage (persistent volumes),
Configuration files of Kubernetes and the other programs of the control plane,
the current status of the Kubernetes cluster including the extensions,
Databases of the configuration, specifically here etcd,
all infrastructure applications that are necessary for the operation of the cluster and the services within it and
the data management of the code and image registries.
Snapshots for running the applications SHOULD also be considered. Snapshots MUST NOT replace backups.
The data protection of a cluster must be defined individually as part of the system architecture and the operating model. The areas of responsibility of the platform (system administration) and application management (specialist administration) must be considered separately.

### Plattform

Backup of the cluster meta database (ETCD)
supplementary backup of all cluster resources in an individual form, since a backup of the ETCD database only allows an 'all-or-nothing' approach for the restore
Backup of selected persistent volumes of platform-specific components, e.g. for the OpenShift internal image registry
Backup of security-related logs such as the OpenShift Audit Log
for other components, it must be defined in the operating concept whether data backup is necessary or whether the loss of data can be tolerated
Examples: Monitoring data or data from the OpenShift internal log aggregation, provided the latter has been forwarded to external data sinks, for example
External tools (e.g. Velero, https://velero.io/) can be used in conjunction with the 'OpenShift APIs for Data Protection' (OADP) (https://github. com/openshift/oadp-operator) or tools from commercial providers can be used.

### Applications

From a platform point of view, relevant data parts of the cluster can be saved
However, the backup of individual fixed memory portions (persistent volumes) of applications can only be carried out from a technical administrative point of view, since sufficient knowledge about the creation of consistent backups is only available from the application point of view
If, from a platform perspective, all persistent volumes are backed up, it must be clearly defined that those responsible for the application are responsible for defining the usability of these backups. In the case of complex applications and/or databases in particular, only a backup mechanism that is self-initiated from the application point of view will be able to create a consistent backup that can be used for restore purposes.


### Snapshot-based backups

The possibility of using snapshots in persistent volumes depends on the storage provider used. As described in the safeguard, snapshots can be a supporting element, but cannot replace backups. The same applies to snapshots that are created in external storage backends by these backends and are therefore independent of OpenShift. With regard to the applications operating on the persistent volumes, back-end backups are also subject to the same consideration as backups from the platform perspective: from the technical perspective of the application, the consistency of these backups is not necessarily guaranteed.
disaster recovery

In the case of complete CD pipelines, disaster recovery can consist of restoring the last data status from the backup and rolling it out via CD pipeline. If such a CD pipeline does not exist, backups of the application resources must be used.

### APP.4.4.M6 Initialization of Pods (S)

If, for example, an application is initialized in the pod at the start, this SHOULD take place in a separate init container. It SHOULD be ensured that the initialization terminates all processes already running. Kubernetes SHOULD ONLY start the other containers after successful initialization.
OpenShift provides the necessary resource configurations via Kubernetes. Kubernetes ensures the (process) dependencies between init containers and "normal" containers of a pod.
The requirement must be implemented by the application development. If several pods have to be initialized, the control can be implemented very easily using operators using the [OPERATOR-TOOLKIT].


### APP.4.4.M7 Separation of Kubernetes-Networks (S)

See also:   Red Hat OpenShift Security Guide "Network Security" (Kap. 6, S. 183ff) 

### APP.4.4.M7 Separation of networks in Kubernetes (S)

The networks for the administration of the nodes, the control plane and the individual networks for the application services SHOULD be separated.
ONLY the network ports of the pods that are necessary for operation SHOULD be released in the networks provided for this purpose. With multiple applications on a Kubernetes cluster, all network connections between the Kubernetes namespaces SHOULD initially be prohibited and only required network connections allowed (whitelisting). The network ports required for the administration of the nodes, the runtime and Kubernetes including its extensions SHOULD ONLY be accessible from the administration network and from pods that require them.
Only selected administrators SHOULD be authorized in Kubernetes to manage the CNI and create or change rules for the network.
The network separation in Kubernetes is described in the Red Hat OpenShift Security Guide in the chapter "Network Security" (chap. 6, p. 183ff).


### APP.4.4.M8 Protection of Kubernetes Configuration Files (S)

The configuration files of the Kubernetes cluster, including all extensions and applications, SHOULD be versioned and annotated.
Access rights to the administration software for the configuration files SHOULD be granted at a minimum. Access rights for reading and writing access to the configuration files of the control plane SHOULD be assigned and restricted with particular care.
OpenShift is fully controlled via Kubernetes resources and custom resources (CR). All CRs that are executed after the initial cluster installation as part of "Day-1" or "Day-2" belong to the configuration files.
These CRs are in system namespaces that only cluster administrators have access to.
If, for example, Git is used to manage the cluster configuration, the versioning is complete and access restrictions must be implemented there.

### APP.4.4.M9 Use of Kubernetes Service-Accounts (S)

Pods SHOULD NOT use the "default" service account. No rights SHOULD be granted to the "default" service account. Pods for different applications SHOULD each run under their own service accounts. Permissions for the service accounts of the application's pods SHOULD be limited to strictly necessary privileges.
Pods that do not require a service account SHOULD not be able to see it and have access to corresponding tokens.
Only pods in the control plane and pods that absolutely need them SHOULD use privileged service accounts.
Automation programs SHOULD each receive their own tokens, even if they use a common service account for similar tasks.
The protective measure is of an organizational nature. It must be taken into account when configuring the deployment:
Creation of a specific service account for the deployment
Definition of an individual, local namespace role for deployment and assignment of the role to the ServiceAccount
This role should be defined in terms of the least privileges principle for the minimum amount of permissions
analogous definition of service accounts, roles and role bindings for automation purposes
Leverage Red Hat Advanced Cluster Security for Kubernetes policies to prevent use of the "default" ServiceAccount.
In the OpenShift platform, the system-side namespaces (control plane and other system components) are only accessible to administrative users. System-side components also run with privileged authorizations if required.
In principle, via [SCC] as a further element of the authorization control, similar to the RBAC model, extended privileges can also be provided by the platform administration on a fine-grained basis for those components that require such extended authorizations. The principle of least privilege should also apply here.

### APP.4.4.M10 Protection of automation processes (S)

All processes of the automation software, such as CI/CD and their pipelines, SHOULD only work with absolutely necessary rights. If different user groups can change the configuration or start pods via the automation software, this SHOULD be done for each group using separate processes that only have the rights required for the respective user group.
OpenShift Container Platform GitOps can be connected to the OpenShift [RBAC]. For this purpose, the SSO is carried out according to the documentation Configuring SSO for Argo CD using Dex.
The ArgoCD groups must be defined accordingly in the configuration as "policy" and kept in sync with the groups of the namespace administrators.


### APP.4.4.M11 Monitoring of the containers (S)

In pods, each container SHOULD define a health check for startup and operation ("readiness" and "liveness"). These checks SHOULD provide information about the availability of the software running in the pod. The checks SHOULD fail if the monitored software cannot perform its tasks properly. Each of these checks SHOULD define a period of time appropriate to the service operated in the pod. Based on these checks, Kubernetes SHOULD delete or restart the pods.
The protective measure is of an organizational nature. It must be implemented during application development and the deployment configuration of the probes.


### APP.4.4.M12 Protection of infrastructure applications

If you have your own registry for images or software for automation, for managing the hard drive, for storing configuration files or similar, you SHOULD at least consider the following:
use of personal and service accounts for access,
encrypted communication on all network ports,
minimal assignment of permissions to users and service accounts,
Logging of changes and regular data backup.
The OpenShift internal registry already fulfills all measures except for the backup with a standard installation. A backup must be set up for this according to APP.4.4.M5.
All operations on the infrastructure applications are secured via the OpenShift API via [RBAC] and are logged in the audit log

### APP.4.4.M13 Automated configuration auditing (H)

There SHOULD be an automatic audit of the settings of the nodes, of Kubernetes and of the pods of the applications against a defined list of allowed settings and against standardized benchmarks.
Kubernetes SHOULD enforce the established rules in the cluster by connecting suitable tools.
In addition to the internal audit log file of the OpenShift cluster, Red Hat Advanced Cluster Management for Kubernetes and Red Hat Advanced Cluster Security for Kubernetes support this measure.
Policies in Red Hat Advanced Cluster Security for Kubernetes monitor the configuration and report deviations from the desired configuration. The policies take effect regardless of the type of configuration change.

### APP.4.4.M14 Use of dedicated nodes (H)

In a Kubernetes cluster, the nodes SHOULD be assigned dedicated tasks and only operate pods that are assigned to the respective task.
Bastion nodes SHOULD handle all incoming and outgoing data connections from applications to other networks.
Management nodes SHOULD run the pods of the control plane and they SHOULD only handle the data connections of the control plane.
If deployed, storage nodes SHOULD only operate the pods of the storage services in the cluster.
In the basic configuration, OpenShift knows master and worker nodes. The division of worker nodes into infra and worker nodes is well documented and makes sense for commercial reasons. If possible, the infranodes should also be divided into several node groups (observability, storage, ...​).
Furthermore, the worker nodes can also be provided specifically for applications. To do this, another label is added to the worker label to separate the generic nodes and the application-specific nodes (a separate label for each application) and the default node selector is expanded to include the generic label. The namespaces with dedicated nodes receive the corresponding additional labels in the namespace's node selector.
The separation takes place via [LABELS] on nodes and corresponding [NODESELECTORS] on the namespace or deployment level.
In addition, a separate bastion host should be used for each cluster in order to create a clear separation between several OpenShift environments and to prevent access being mixed up.
INFO: With Kubernetes, this hard separation is a hard security requirement since OpenShift's strict SE-Linux-based sandboxing does not exist. When planning, it should be considered whether this hard sandbox separation already fulfills the necessary separation of the applications.

## APP.4.4.M15 Separation of applications at node and cluster level (H)

Applications with a very high protection requirement SHOULD use their own Kubernetes cluster or dedicated nodes that are not available for other applications.
A simple separation of the applications is already done by APP.4.4.M1 or APP.4.4.M14. If applications have their own clusters installed, Red Hat Advanced Cluster Management for Kubernetes helps enforce the necessary configurations. Automation via ACM creates the necessary cluster configuration for applications in a simple and reproducible manner.

## APP.4.4.M16 Use of operators (H)

The automation of operational tasks in operators SHOULD be used for particularly critical applications and the programs of the control plane.
OpenShift relies consistently on the application of the concept of Kubernetes operators. The platform itself is operated and managed 100% by operators, i.e. all internal platform components - both the central management plan and other platform components - are rolled out and managed by operators.
Application-specific operators must be considered as part of application development and deployment.

## APP.4.4.M17 Attestation of nodes (H)

Nodes SHOULD send a cryptographically and preferably TPM verified secure status report to the control plane. The control plane SHOULD ONLY include nodes in the cluster that have successfully demonstrated their integrity.
TODO OpenShift uses its own internal Certificate Authority (CA). The nodes communicate with the management plane via TLS connections that are secured with node-specific certificates from this CA. The internal CA is created during the installation (bootstrap) process and managed in the management plane.
There is no protection via a TPM.


## APP.4.4.M18 Use of micro-segmentation

The pods SHOULD only be able to communicate with each other within a Kubernetes namespace via the necessary network ports. There SHOULD be rules within the CNI that prohibit all but the network connections necessary for operation within the Kubernetes namespace. These rules SHOULD clearly define the source and destination of the connections, using at least one of the following criteria: service name, metadata ("labels"), the Kubernetes service accounts, or certificate-based authentication.
All criteria used to designate this connection SHOULD be secured in such a way that they can only be changed by authorized persons and administrative services.
Building block APP.4.4.M7 describes a separation at the namespace level. Removing the network policy for pods in the same namespace allows additional network policies to set up the necessary micro-segmentation within a namespace.

## APP.4.4.M19 High Availability Kubernetes (H)

Operations SHOULD be structured in such a way that if one location fails, the clusters and thus the applications in the pods either continue to run without interruption or can be restarted at another location in a short time.
For the restart, all necessary configuration files, images, user data, network connections and other resources required for operation, including the hardware required for operation, SHOULD already be available at this location.
For the uninterrupted operation of the cluster, the control plane of Kubernetes, the infrastructure applications of the clusters and the pods of the applications SHOULD be distributed over several fire compartments based on location data of the nodes in such a way that the failure of a fire compartment does not lead to the failure of the application.
Openshift supports topology labels to separate multiple fire compartments (so-called "Failure Zones"). Setting up complete failover clusters is supported by Red Hat Advanced Cluster Management for Kubernetes to distribute the necessary applications. Red Hat OpenShift Data Foundation can be used for the necessary data transfer.
Failures of individual nodes or entire fire compartments do not lead to a cluster or application failure as long as the applications and infrastructure services have been distributed to the fire compartments accordingly.

## APP.4.4.M20 Encrypted data storage for pods (H)

The file systems with the persistent data of the control plane (here especially etcd) and the application services SHOULD be encrypted.
Full Disk Encryption can be used as a basis (Red Hat OpenShift Security Guide, p. 53ff). When using Red Hat OpenShift Data Foundation, the volumes can also be encrypted there.

### APP.4.4.M21 Periodic restart of pods (H)

If there is an increased risk of external interference and a very high protection requirement, pods SHOULD be stopped and restarted regularly. No pod SHOULD run longer than 24 hours. The availability of the applications in the pod SHOULD be ensured.
A forced regular restart of pods is done via individual Kubernetes-based mechanisms (e.g. Kubernetes CronJobs to delete the pod). Kubernetes and OpenShift provide sufficient mechanisms to ensure the availability of the applications. An application-side view of long-running processes and forced restarts of pods is required.


### Selection of Policy-Enforcement-Points (Technologies)

Technology selection is a process where you might not only match the Compliance-Goals but also the Skills which are available in your Organization or subjective preferences based on personal investigation. Further we recommend to not start with all possible options, start with maybe 10 to 15 (high-prio) policies, monitor the results and then configure additional checks.


After an initial review of possible options we selected the following technologies:

Compliance-Operator
Kyverno
ACM-Config-Policies

We could extend this anytime with other technologies like ACS, Sandboxed-Containers or Security-Operator but we also need to consider that we only use what we need to not overload the Kubernetes-Clusters with too many checks which might affect the performance.


Mapping the resources to the standard

This step in the process might be the most important and requires a deep study of the documents which are provided by the Governance-Authority

If you would - as an example - use NIST-8053 standard  you can already follow the Structure of https://github.com/stolostron/policy-collection/community and select examples which require only some adjustment so that they are fitting to your environment.
 

After a review we decided to use the following examples to implement the standard:

Kyverno

Kyverno-Validate for checking that: PU and memory resource requests and limits are required
Kyverno-Mutate for achieving that every namespace has a clear label who is responsible for this
Kyverno-Generate for achieving RBAC-requirements

Compliance Operator

Compliance-Operator-Installation
Compliance-Operator-Configuration for selecting the right compliance-profile
Compliance-Operator-Remediation for automatically applying the necessary changes based on previous Compliance-Scans.

ACM and OpenShift Hardening

ETCD-Encryption ()
ETCD-Backup
Checks that no Non-Compliant-Policies exist.
Audit is enabled and configured properly
Audit-logs are stored into a Systems which have the capability for further analysis/storage
FIPS is enabled
We control the Kubeadm-User of the different Kubernetes-Clusters has been disabled or a password which can be centrally controlled
Secrets are Stored in a Centralized-Way and securely distributed to Clusters.
The Storage-Class should be configured with limits regarding it’s usage. 


### Concrete Mapping-Example

We identified that ETCD-Encryption could be mapped to:

APP.4.4.A20 Encrypted data storage for pods (H)

The file systems with the persistent data of the control plane (here especially etcd) and the
Application services SHOULD be encrypted.

We will map this to the policy-fields:

Standards: BSI-Grundschutz
Controls: Security
Categories: APP.4.4.A20


GIT would probably have to be considered depending on the intended use (registry see APP.4.4.A12, for version management see CON.8.A10)
Please note that there is a direct reference to operators (APP.4.4.A16)
RBAC can be found in the requirements for increased protection requirements (SYS.1.6.A21), multitenancy is treated rather inconsistently as client capability. Sometimes as a threat ("lack of multi-client capability"), sometimes as a requirement, and sometimes indirectly via the topic of isolation, depending on the module. For containers, more about the requirements for insulation (SYS.1.6.A3, SYS.1.6.A26, APP.4.4.A4).

For Kubernetes there is a specific requirement for data backup in the Kubernetes cluster (APP.4.4.A5).


### Configuring the Standard using Policy-Generator

Policy-Generator is a powerful tool for creating policies based on input files which even could be 
Gatekeeper or Kyverno. See here for some comprehensive introduction

Please also review the following link for checking the schema of a Policy-Generator config-file:

https://github.com/stolostron/policy-generator-plugin/blob/main/docs/policygenerator-reference.yaml

You see how flexible you can set both global values (in our case Standard: BSI-Grundschutz) and Policy-specific values.

Based on that we configure the whole standard in the following Configuration-File:

https://github.com/ch-stark/maptoseveralstandards/blob/main/config/bsigrundschutz/policyGenerator_bsigrundschutz.yaml


### Monitoring the results
    
The ACM-UI is the first place to look but we have also advanced options like sending 
information to Ansible or to RHACM’s Observability-Framework which in turn can export the
information or send it to other Systems like email, slack, service now or you could even  
invoke Ansible from where.


### Summary

We can use the same procedure to implement other Compliance-Standards based on the needs of your Organization. You can 100% reuse the provided input files, make a Technology selection, select the required input files and generate the Policies.
