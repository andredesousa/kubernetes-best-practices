# Kubernetes Best Practices

Kubernetes, also known as K8s, is an open-source system for automating deployment, scaling, and management of containerized applications.
These tips are based on books, articles and professional experience.

## Table of Contents

1. [Use the latest version](#use-the-latest-version)
2. [Use managed clusters](#use-managed-clusters)
3. [Use multiple nodes](#use-multiple-nodes)
4. [Isolate nodes](#isolate-nodes)
5. [Use namespaces](#use-namespaces)
6. [Use role-based access control](#use-role-based-access-control)
7. [Use third-party authentication provider](#use-third-party-authentication-provider)
8. [Use official images](#use-official-images)
9. [Use smaller images](#use-smaller-images)
10. [Update images frequently](#update-images-frequently)
11. [Use a private registry](#use-a-private-registry)
12. [Use declarative YAML files](#use-declarative-yaml-files)
13. [Use network policies](#use-network-policies)
14. [Use a firewall](#use-a-firewall)
15. [Use process whitelisting](#use-process-whitelisting)
16. [Avoid privileged containers](#avoid-privileged-containers)
17. [Use a read-only filesystem in containers](#use-a-read-only-filesystem-in-containers)
18. [Set resource requests and limits](#set-resource-requests-and-limits)
19. [Use annotations and labels](#use-annotations-and-labels)
20. [Use readiness and liveness probes](#use-readiness-and-liveness-probes)
21. [Use logging and monitoring tools](#use-logging-and-monitoring-tools)
22. [Automatically roll deployments](#automatically-roll-deployments)
23. [Use autoscaling](#use-autoscaling)
24. [Turn on audit logging](#turn-on-audit-logging)
25. [Monitor control plane components](#monitor-control-plane-components)
26. [Monitor disk usage](#monitor-disk-usage)
27. [Monitor network traffic](#monitor-network-traffic)
28. [Have an alert system](#have-an-alert-system)
29. [Externalise all configurations](#externalise-all-configurations)
30. [Lint manifest files](#lint-manifest-files)
31. [Be proactive using Policy-as-Code](#be-proactive-using-policy-as-code)
32. [Prefer stateless applications](#prefer-stateless-applications)
33. [Avoid databases on Kubernetes](#avoid-databases-on-kubernetes)
34. [Use a version control system](#use-a-version-control-system)
35. [Use GitOps](#use-gitops)
36. [Use Helm](#use-helm)
37. [Lock down Kubelet](#lock-down-kubelet)
38. [Protect etcd](#protect-etcd)
39. [Restrict API access](#restrict-api-access)
40. [Restrict SSH access](#restrict-ssh-access)
41. [Use Deployments instead of Pods](#use-deployments-instead-of-pods)
42. [Define the risk tolerance in code](#define-the-risk-tolerance-in-code)
43. [Run more than one replica](#run-more-than-one-replica)
44. [Set disruption budgets](#set-disruption-budgets)
45. [Mount secrets as volumes](#mount-secrets-as-volumes)
46. [Store secrets encrypted](#store-secrets-encrypted)
47. [Set graceful shutdown](#set-graceful-shutdown)
48. [Application logs to stdout and stderr](#application-logs-to-stdout-and-stderr)
49. [Avoid sidecars for logging](#avoid-sidecars-for-logging)
50. [Include PVCs in Pod configuration](#include-pvcs-in-pod-configuration)

## Use the latest version

Ensure that your Kubernetes cluster is up to date.
In addition to updates and additional features, the latest release will have patches to previous version security issues.
This is critical for mitigating many of the vulnerabilities that could affect your cluster.
Older versions also don't get as much support from the Kubernetes community.
Migrating to a new version should be treated with caution however as certain features can be depreciated, as well as new ones added.

## Use managed clusters

Unless you have a large engineering department with teams already used to running open-source software as highly available services, then a managed Kubernetes service, either in the cloud or on-premise, will substantially lower its complexity by having fewer components to administer and upgrade.
Depending on the type of hosted service you choose, your team won't have to deal with manually provisioning the [control plane components](https://kubernetes.io/docs/concepts/overview/components/#control-plane-components), adding nodes to expand your cluster, or configuring the pods that support your containers.

## Use multiple nodes

Running Kubernetes on a single node is not a good idea if you want to build in fault tolerance.
Multiple nodes should be employed in your cluster so workloads can be spread between them.
A Kubernetes cluster that handles production traffic should have a minimum of three nodes because if one node goes down, both an [etcd](https://kubernetes.io/docs/concepts/overview/components/#etcd) member and a control plane instance are lost, and redundancy is compromised.

## Isolate nodes

[Kubernetes nodes](https://kubernetes.io/docs/concepts/architecture/nodes/) must be on a separate network and should not be exposed directly to public networks.
If possible, you should even avoid direct connections to the general corporate network.
This is only possible if Kubernetes control and data traffic are isolated. Otherwise, both flow through the same pipe, and open access to the data plane implies open access to the control plane.
Ideally, nodes should be configured with an ingress controller, set to only allow connections from the master node on the specified port through the network access control list.

## Use namespaces

In Kubernetes, [namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) provides a mechanism for isolating groups of resources within a single cluster.
Namespaces are intended for use in environments with many users spread across multiple teams, or projects.
For example, you should create different namespaces for development, testing and production applications.
This way, the developer having access to only the development namespace won't be able to make any changes in the production namespace, even by mistake.
If you do not do this separation, there is a high chance of accidental overwrites by well-meaning teammates

## Use role-based access control

[Role-Based Access Control](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) (RBAC) is a method of regulating access to computer or network resources based on the roles of individual users within your organization.
RBAC can help you define who has access to the Kubernetes API and what permissions they have.
When using RBAC, prefer namespace-specific permissions instead of cluster-wide permissions.
Even when debugging, do not grant cluster administrator privileges.
It is safer to allow access only when necessary for your specific situation.

## Use third-party authentication provider

It is recommended to integrate Kubernetes with a third-party authentication provider.
This provides additional security features such as multi-factor authentication, and ensures that [kube-apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) does not change when users are added or removed.
If possible, make sure that users are not managed at the API server level.
You can use OAuth 2.0 connectors like [Dex](https://dexidp.io/).
Alternatively, cloud providers like AWS, Azure or Google Cloud Platform provide OAuth to authenticate with the Kubernetes API server.

## Use official images

[Docker Official Images](https://hub.docker.com/search?q=&type=image&image_filter=official) are a curated set of Docker repositories hosted on Docker Hub.
Official images are created with the best practices that affect the versatility, security, and efficiency of your container.
Docker Official Images are some of the most popular on Docker Hub.
Docker, Inc. sponsors a dedicated team that is responsible for reviewing and publishing all content in the Docker Official Images.
This team works in collaboration with upstream software maintainers, security experts, and the broader Docker community.

## Use smaller images

If the image is based on a full-blown OS distribution like Ubuntu or CentOS, you will have a bunch of tools already packaged in the image.
Instead, use smaller container images such as [Alpine](https://hub.docker.com/_/alpine) images which are several times smaller.
In comparison by using smaller images with leaner OS distributions, which only bundle the necessary system, you need less storage space, minimize the attack surface, and ensure you create more secure containers.

## Update images frequently

Prefer images that are updated frequently. As new security vulnerabilities are discovered continuously, it is a general security best practice to stick to the latest security patches.
There is no need to always go to the latest version, which might contain breaking changes, but define a versioning strategy.
You can stick to stable or long-term support versions, which deliver security fixes soon and often.
Also, [scan your images regularly](https://docs.docker.com/develop/security-best-practices/#check-your-image-for-vulnerabilities) for malware, misconfigurations and other risks that could lead to a security breach.

## Use a private registry

Container registries are highly convenient, letting you download container images at the click of a button, or automatically as part of development and testing workflows.
However, together with this convenience comes a security risk.
There is no guarantee that the image you are pulling from the registry is trusted.
It may unintentionally contain security vulnerabilities or may have intentionally been replaced with an image compromised by attackers.
The solution is to use a private registry deployed behind your own firewall, to reduce the risk of tampering.
[Nexus](https://www.sonatype.com/products/repository-oss) and [Artifactory](https://jfrog.com/artifactory/) are alternatives to Docker Hub.

## Use declarative YAML files

Replace using imperative [kubectl](https://kubernetes.io/docs/reference/kubectl/) commands like `kubectl run` with writing declarative YAML files.
You can then add them to your cluster by using the `kubectl apply` command.
A declarative approach lets you specify your desired state, and Kubernetes figures out how to attain it.
YAML files allow you to store and version all your objects along with your code.
You can easily roll back deployments if things go wrong. Just restore an earlier YAML file and reapply it.
In addition, this model ensures your team can see the cluster's status and changes made to it over time.

## Use network policies

Use [network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/) to restrict access to services within a Kubernetes cluster.
By default, all containers can talk to each other in the network, something that presents a security risk if malicious actors gain access to a container, allowing them to traverse objects in the cluster.
Network policies can control traffic at the IP and port level, similar to the concept of security groups in cloud platforms to restrict access to resources.
Typically, all traffic should be denied by default, then allow rules should be put in place to allow required traffic.
You'll find several examples of network policies in the [Kubernetes Network Policy Recipes](https://github.com/ahmetb/kubernetes-network-policy-recipes/) repository.

## Use a firewall

You should put a firewall in front of your cluster to restrict requests to the API server from the outside world.
IP addresses should be whitelisted, and open ports restricted.
You can use regular firewalling rules or port firewalling rules. If you are using something like [Azure Kubernetes Service](https://azure.microsoft.com/products/kubernetes-service), [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine), or [Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/), then you can use the features provided by these public cloud providers.

## Use process whitelisting

Process whitelisting is an effective way to identify unexpected running processes.
First, observe the application over a period of time to identify all processes running during normal application behaviour.
Then use this list as your whitelist for future application behaviour.
Suppose an attacker succeeds in getting access to your cluster and begins running a malicious process.
In that case, you'll easily identify the user since it's a new non-whitelisted process.

## Avoid privileged containers

Ensuring that a container can perform only a very limited set of operations is vital for production deployments.
This is possible thanks to the use of non-root containers, which are executed by a user different from root.
You can restrict the container capabilities to the minimal required set using [securityContext.capabilities.drop](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#set-capabilities-for-a-container) option.
That way, in case your container is compromised, the range of action available to an attacker is limited.

## Use a read-only filesystem in containers

A read-only root filesystem helps to enforce an immutable infrastructure strategy.
The container should only write on mounted volumes that can persist, even if the container exits.
Using an immutable root filesystem and a verified boot mechanism prevents against attackers from "owning" the machine through permanent local changes.
An immutable root filesystem can also prevent malicious binaries from writing to the host system.

## Set resource requests and limits

[Resource requests and limits](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) should be set to avoid a Pod starting without the required resources assigned, or the Pod running out of available resources.
Without limits, Pods can utilize more resources than required, causing the total available resources to be reduced which may cause a problem with other applications on the cluster.
Nodes may crash, and new Pods may not be able to be placed corrected by the scheduler.
When defining CPU and memory limits for either requests or limits, millicores are typically used for CPUs and mebibytes or megabytes for memory.

## Use annotations and labels

As the number of objects in your cluster grows, it gets more difficult to find and organize them.
You can use either labels or annotations to attach metadata to Kubernetes objects.
[Labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) can be used to select objects and to find collections of objects that satisfy certain conditions.
In contrast, [annotations](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/) are not used to identify and select objects.
The metadata in an annotation can be small or large, structured or unstructured, and can include characters not permitted by labels.

## Use readiness and liveness probes

Many applications running for long periods of time eventually transition to broken states and cannot recover except by being restarted.
[Readiness and liveness](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) probes are essentially health checks.
The [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/) uses liveness probes to know when to restart a container and readiness probes to know when a container is ready to start accepting traffic.
The probe pings the pod for a response.
No response means your app is not running on that particular pod, triggering the probe to launch a new pod and run the application there.

## Use logging and monitoring tools

You should set up automated monitoring to help you make sense of Kubernetes alerts.
Monitoring your cluster is critical for identifying issues and controlling resource usage.
Issues with your cluster can worsen your product's performance, increase your operational costs, and in the worst case, cause outages.
Monitoring will allow you to identify these problems more quickly and understand their causes.
In order to gather this information, you would commonly deploy logging stacks like ELK ([ElasticSearch](https://www.elastic.co/elasticsearch/), [Logstash](https://www.elastic.co/logstash/), and [Kibana](https://www.elastic.co/kibana/)) and monitoring tools like [Prometheus](https://prometheus.io/).

## Automatically roll deployments

It is common to have ConfigMaps or Secrets mounted to containers.
Although the deployments and container images change with new releases, the ConfigMaps or Secrets do not change frequently.
[Reloader](https://github.com/stakater/Reloader) can watch changes in ConfigMap and Secret and do rolling upgrades on Pods with their associated DeploymentConfigs, Deployments, Daemonsets Statefulsets and Rollouts.
Alternatively, [Helm sha256sum function](https://helm.sh/docs/howto/charts_tips_and_tricks/#automatically-roll-deployments) can be used to ensure a deployment's annotation section is updated if another file changes.

## Use autoscaling

Kubernetes lets you automate many management tasks, including provisioning and scaling.
Instead of manually allocating resources, you can create automated processes that save time, allow you to respond quickly to spikes in demand, and conserve costs by scaling down when resources are not needed.
Where it is appropriate, autoscaling can be employed to dynamically adjust the number of pods ([Horizontal Pod Autoscaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)), the amount of resources consumed by the pods ([Vertical Pod Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler)), or the number of nodes in the cluster ([Cluster Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler)), depending on the demand for the resources.

## Turn on audit logging

Make sure that [audit logging](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/) is enabled and you are monitoring unusual or unwanted API calls, especially authentication failures.
Enabling audit logs is a standard security measure that can also be useful in day-to-day operations.
With this feature, each action by an administrator is logged in a file for future reference.
Tracing configuration changes to individual administrators in a multi-user platform can help track down and correct problematic usage patterns.

## Monitor control plane components

The control plane is the brain and heart of Kubernetes, It includes [kube-apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/), [etcd](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/), [kube-scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/), [controller-manager](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/), [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/), [kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/) and [DNS](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/). Monitoring the control plane helps identify vulnerabilities within the cluster.
It is recommended using a robust, continuous, and automated Kubernetes monitoring tools rather than managing this manually.
You'll also be able to monitor resource usage so you can optimize performance and costs.

## Monitor disk usage

Pods running on Kubernetes may claim a Persistent Volume to store data that last between Pod restarts.
This volume is usually of limited size, so you need to monitor its storage and alert for low free space.
You should regularly monitor all volumes associated with your cluster, as well as the root file system.
With alert monitoring, you'll be able to take corrective actions either by scaling or freeing disk space when needed.

## Monitor network traffic

Containerized applications generally make extensive use of cluster networks.
Observe active network traffic and compare it to the traffic allowed by Kubernetes network policy, to understand how your application interacts and identify anomalous communications.
At the same time, if you compare active traffic to allowed traffic, you can identify network policies that are not actively used by cluster workloads.
This information can be used to further strengthen the allowed network policy, removing unneeded connections to reduce the attack surface.

## Have an alert system

Alerts allow you to identify problems in the system moments after they occur.
By quickly identifying unintended changes to the system, you can minimize service disruptions.
Alerts consists in alert rules and notification channel.
Configure alerts to quickly get notified when the values exceed the expected thresholds.
Use meaningful dashboards to explore the evolution of metrics and correlate with changes in other metrics and events happening in your system.

## Externalise all configurations

Configurations should be maintained outside the application code.
This has several benefits. First, changing the configuration does not require recompiling the application.
Second, the configuration can be updated when the application is running.
Third, the same code can be used in different environments.
In Kubernetes, the configuration can be saved in ConfigMaps, which can then be mounted into containers as volumes are passed in as environment variables. Save only non-sensitive configuration in ConfigMaps.
For sensitive information (such as credentials), use the Secret resource.

## Lint manifest files

Use a linter to avoid common mistakes and establish best practice guidelines that engineers can follow in an automated way.
[KubeLinter](https://github.com/stackrox/kube-linter) and [Kubeval](https://github.com/instrumenta/kubeval/) can be used to parse Kubernetes YAML files and Helm charts, and are often used locally as part of a development workflow as well as in CI pipelines.
You can supplement this analysis with vulnerability scanning.
[kubeaudit](https://github.com/Shopify/kubeaudit) can be used to audit a manifest file before the manifest resources are deployed to the cluster.

## Be proactive using Policy-as-Code

You can take advantage of policy-as-code tools to scan configuration data and check if it conforms to the codified policies. [Kyverno](https://kyverno.io/) is a policy engine designed for Kubernetes.
Kyverno allows cluster administrators to manage environment specific configurations independently of workload configurations and enforce configuration best practices for their clusters.
Alternatively, you can use other tools like [Terrascan](https://runterrascan.io/) to scan configurations and flag any non-compliance for Infrastructure as Code.

## Prefer stateless applications

In general, stateless apps are easier to manage than stateful apps. With a stateless backend, development teams can make sure there aren't long-running connections that make scaling more challenging.
Using stateless also enables developers to deploy applications more efficiently with zero downtime.
Stateless applications make it easier to migrate and scale as and when required according to business needs.
Ensure a smooth user experience by preserving the cluster for differentiated services and storing data separately.

## Avoid databases on Kubernetes

Databases require high input and output (I/O) throughput, which means that they need to rapidly read and write vast amounts of data to storage volumes.
This means that databases can conflict with other applications when reading and writing to storage and experience performance bottlenecks. Kubernetes is ideal for hosting stateless applications that can scale horizontally via replication.
Database workloads don't fit that workload profile and are a poor candidate as a Kubernetes workload.
Dedicated virtual machines or managed database services offered by public cloud providers are the best solutions for delivering a database service.

## Use a version control system

Store all config files, such as deployment, services, and ingress ones in a version control system.
Version control is an important part of [Infrastructure as Code](https://www.redhat.com/en/topics/automation/what-is-infrastructure-as-code-iac) (IaC), and your configuration files should be under source control just like any other software source code file.
Doing this before pushing your code to a cluster enables you to track source code changes and who made them.
Whenever necessary, you can quickly roll back the change, re-create, or restore your cluster to ensure stability and security.

## Use GitOps

[GitOps](https://www.gitops.tech/) is a way of implementing [Continuous Deployment](https://www.atlassian.com/continuous-delivery/continuous-deployment) for cloud native applications.
It focuses on a developer-centric experience when operating infrastructure, by using tools developers are already familiar with, including Git and Continuous Deployment tools.
GitOps works by using Git as a source of truth for declarative infrastructure and applications.
Using a GitOps can help improve productivity by speeding up application development and lowering deployment times.
It can also enhance error traceability and automate CI/CD workflows.

## Use Helm

[Helm](https://helm.sh/) is a package manager for Kubernetes applications, providing mechanisms to template and group Kubernetes manifests as versioned packages, and simplifies the installation of new applications.
Helm plays a vital role in making Kubernetes applications easier to build and test in a continuous integration and delivery pipeline.
Alternatively, you can use [Kustomize](https://kustomize.io/), a template-free way to customize your application configuration.

## Lock down Kubelet

The [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/) is an agent running on each node, which interacts with container runtime to launch pods and report metrics for nodes and pods.
If an unauthorized user gains access to this API and can run code on the cluster, they can compromise the entire cluster.
You should disable anonymous access with `--anonymous-auth=false` so that unauthenticated requests get an error response.
Set `--authorization` mode to a value other than AlwaysAllow to verify that requests are authorized and set `--read-only-port=0` to close read-only ports.

## Protect etcd

[Kubernetes uses etcd](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/) to store cluster state and its secrets.
[etcd](https://etcd.io/) is a strongly consistent, distributed key-value store that provides a reliable way to store data that needs to be accessed by a distributed system or cluster of machines.
It is a sensitive resource and an attractive target for attackers.
If unauthorized users gain access to etcd they can take over the entire cluster.
Read access is also dangerous because malicious users can use it to elevate privileges.
It is recommended to protect etcd with TLS, firewall and encryption.

## Restrict API access

Most cloud-based Kubernetes implementations already limit access to the Kubernetes API of your cluster via the use of Identity and Access Management (IAM), Role-Based Access Control (RBAC), or Active Directory (AD).
If your cluster doesn't already use one of these methods, you can typically configure one by using open-source projects to deal with different authentication methods.
Additionally, access to the API should also be limited by the IP address, and access should only be provided to the trustworthy IPs.

## Restrict SSH access

It's an excellent practice to disallow SSH access to the Kubernetes nodes.
This will lower the risk of unauthorized access to host resources on the cluster.
Configure your nodes via your cloud provider to block access to port 22 except via your organization's VPN or a bastion host.
You can also take advantage of Kubernetes authorization plugins to restrict user access to resources.
You can even define access control rules for containers, namespaces, and operations.

## Use Deployments instead of Pods

A single pod should never be run individually. Avoid using naked pods as much as possible.
To improve fault tolerance, instead, they should always be part of a Deployment, DaemonSet, ReplicaSet or StatefulSet.
In the event of a node failure, naked pods can't be rescheduled because they're not bound to a Deployment or ReplicaSet.
Pods can then be deployed across nodes using anti-affinity rules in your deployments to avoid all pods being run on a single node, which may cause downtime if it was to go down.
Deploying is almost always more efficient than creating pods directly.

## Define the risk tolerance in code

Kubernetes functionality like [disruption budgets](https://kubernetes.io/docs/tasks/run-application/configure-pdb/), [Quality of Service](https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/) (QoS), [topology spread constraints](https://kubernetes.io/docs/concepts/scheduling-eviction/topology-spread-constraints/), and [anti-affinity rules](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity) give you control to define your tolerance to different kinds of application failure modes.
These control features present many configuration options for managing workload placement.
For example, you can prevent two pods from running on the same cluster node to protect against a single node outage which can lead to application downtime.
You can also ensure that each cluster node gets one and only one pod to host a utility such as virus protection software.

## Run more than one replica

Pods are ephemeral by nature, if a Pod (or the node it executes on) fails, Kubernetes can automatically create a new replica of that Pod to continue operations.
Keep in mind that Pod eviction from a node can happen at any time, and if only 1 replica exists, this would cause a service outage equivalent to the time it takes for a Pod to become available.
Running more than one instance of your Pods guarantees that deleting a single Pod won't cause downtime.
Even if you run several copies of your Pods, there are no guarantees that losing a node won't take down your service.
You should apply anti-affinity rules to your Deployments so that Pods are spread across all the nodes of your cluster.

## Set disruption budgets

When a node is drained, all the Pods on that node are deleted and rescheduled.
But suppose you have a heavy load, and you can't afford to lose over 50% of your Pods.
In that case, you'll want to define a [Pod Disruption Budget](https://kubernetes.io/docs/tasks/run-application/configure-pdb/) to protect the Deployments from unexpected events that could cause unavailability.
Doing this instructs Kubernetes to always prevent the drain event from deteriorating availability further when the final state results in less than the number of Pods you've specified for that Deployment.

## Mount secrets as volumes

Secrets can be mounted as volumes or exposed as environment variables and used by a container in a Pod to interact with external systems on your behalf.
It is reasonably common for application code to log out its environment (particularly in the event of an error).
This will include any secret values passed in as environment variables, so secrets can easily be exposed to any user or entity who has access to the logs.
The content of Secret resources should be mounted into containers as volumes rather than passed in as environment variables.

## Store secrets encrypted

Secrets, like keys, passwords, tokens, and other configuration values, must be stored correctly.
If our Kubernetes cluster is compromised, the Secrets must remain secure.
[Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) are encoded in base64 format and stored as plain text in [etcd](https://etcd.io/).
Kubernetes does not provide robust mechanisms to encrypt, manage, and share secrets across a Kubernetes cluster.
A common approach to getting more secure secret management on Kubernetes is to introduce an external secret management solution, such as [Vault](https://www.vaultproject.io/) or [Sealed Secrets](https://sealed-secrets.netlify.app/).

## Set graceful shutdown

In a Dockerized runtime like Kubernetes, containers are born and die frequently.
This happens not only when errors are thrown but also for good reasons like relocating containers, replacing them with a newer version and more. It's achieved by sending a notice SIGTERM signal to the process with a 30 second grace period.
If the timeout period passes, [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/) sends a SIGKILL signal, which causes the process to be stopped immediately.
Make sure the app is handling the ongoing requests and clean-up resources in a timely fashion.
You can always set a `terminationGracePeriodSeconds` on your pods to terminate containers with a specific grace period.

## Application logs to stdout and stderr

The standard output stream (stdout) is where you send application logs, and the standard error stream (stderr) is where you send error logs.
Whenever an app writes to stdout or stderr, a container engine, like Docker, records, redirects, and stores the file in JSON format.
Yet containers, pods, and nodes in Kubernetes are highly dynamic entities.
These require consistent and persistent logs. So, it's best to keep cluster-wide logs on different backend storage such as the ELK Stack ([ElasticSearch](https://www.elastic.co/elasticsearch/), [Logstash](https://www.elastic.co/logstash/), and [Kibana](https://www.elastic.co/kibana/)).

## Avoid sidecars for logging

If you wish to apply log transformations to an application with a non-standard log event model, you may want to use a sidecar container.
The sidecar approach deploys a separate logging agent for each Pod, which is relatively more resource intensive, but more flexible and multi-tenant isolated.
However, if you have control over the application, you could output the right format.
You could save on running an extra container for each Pod in your cluster.

## Include PVCs in Pod configuration

A  [PersistentVolume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) (PV) is an object that allows pods to access persistent storage on a storage device, defined via a [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/).
Unlike regular volumes, which are transient in nature, PVs are persistent, supporting stateful application use cases.
When creating a PersistentVolume, always configure a PersistentVolumeClaim (PVCs) as part of the container configuration.
Avoid including PVs in container configuration as this strongly pairs a container to a particular volume.
Always specify a default StorageClass, otherwise PVCs without a specific class will fail.

## Bibliography

- [7 Kubernetes Best Practices](https://www.plural.sh/blog/7-kubernetes-best-practices/)
- [10 Kubernetes Best Practices to Get You Started](https://www.densify.com/kubernetes-tools/kubernetes-best-practices/)
- [15 Kubernetes Best Practices Every Developer Should Know](https://spacelift.io/blog/kubernetes-best-practices)
- [20 Kubernetes Best Practices](https://www.eweek.com/cloud/kubernetes-best-practices/)
- [Best Practices in Kubernetes Security](https://www.okteto.com/blog/best-practices-in-kubernetes-security/)
- [Kubernetes production best practices](https://learnk8s.io/production-best-practices)
- [Kubernetes Security Best Practices](https://www.aquasec.com/cloud-native-academy/kubernetes-in-production/kubernetes-security-best-practices-10-steps-to-securing-k8s/)
- [Kubernetes Best Practices](https://www.opsramp.com/guides/why-kubernetes/kubernetes-best-practices/)
- [Kubernetes Best Practices For 2023](https://www.cloudzero.com/blog/kubernetes-best-practices)
