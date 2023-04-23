# Kubernetes Best Practices

## Table of Contents

1. [Use the latest version]()
2. [Use managed clusters]()
3. [Use namespaces]()
4. [Use role-based access control]()
5. [Use third-party authentication provider]()
6. [Use official images]()
7. [Use smaller images]()
8. [Update images frequently]()
9. [Use a private registry]()
10. [Use declarative YAML files]()
11. [Avoid privileged containers]()
12. [Set resource requests and limits]()
13. [Use annotations and labels]()
14. [Use readiness and liveness probes]()
15. [Use logging and monitoring tools]()
16. [Automatically roll deployments]()
17. [Turn on audit logging]()
18. [Have an alert system]()
19. [Lint manifest files]()
20. [Use a version control system]()
21. [Use GitOps]()
22. [Use Helm]()
23. [Isolate Kubernetes nodes]()
24. [Lock down Kubelet]()
25. [Protect etcd]()

## Use the latest version

Ensure that your Kubernetes cluster is up to date.
In addition to updates and additional features, the latest release will have patches to previous version security issues.
This is critical for mitigating many of the vulnerabilities that could affect your cluster.
Older versions also don't get as much support from the Kubernetes community.
Migrating to a new version should be treated with caution however as certain features can be depreciated, as well as new ones added.

## Use managed clusters

Unless you have a large engineering department with teams already used to running open-source software as highly available services, then a managed Kubernetes service, either in the cloud or on-premise, will substantially lower its complexity by having fewer components to administer and upgrade.
Depending on the type of hosted service you choose, your team won't have to deal with manually provisioning the control plane components, adding nodes to expand your cluster, or configuring the pods that support your containers.

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

## Avoid privileged containers

Ensuring that a container can perform only a very limited set of operations is vital for production deployments.
This is possible thanks to the use of non-root containers, which are executed by a user different from root.
You can restrict the container capabilities to the minimal required set using [securityContext.capabilities.drop](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#set-capabilities-for-a-container) option.
That way, in case your container is compromised, the range of action available to an attacker is limited.

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

## Turn on audit logging

Make sure that [audit logging](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/) is enabled and you are monitoring unusual or unwanted API calls, especially authentication failures.
Enabling audit logs is a standard security measure that can also be useful in day-to-day operations.
With this feature, each action by an administrator is logged in a file for future reference.
Tracing configuration changes to individual administrators in a multi-user platform can help track down and correct problematic usage patterns.

## Have an alert system

Alerts allow you to identify problems in the system moments after they occur.
By quickly identifying unintended changes to the system, you can minimize service disruptions.
Alerts consists in alert rules and notification channel.
Configure alerts to quickly get notified when the values exceed the expected thresholds.
Use meaningful dashboards to explore the evolution of metrics and correlate with changes in other metrics and events happening in your system.

## Lint manifest files

Use a linter to avoid common mistakes and establish best practice guidelines that engineers can follow in an automated way.
[KubeLinter](https://github.com/stackrox/kube-linter) and [Kubeval](https://github.com/instrumenta/kubeval/) can be used to parse Kubernetes YAML files and Helm charts, and are often used locally as part of a development workflow as well as in CI pipelines.
You can supplement this analysis with vulnerability scanning.
[kubeaudit](https://github.com/Shopify/kubeaudit) can be used to audit a manifest file before the manifest resources are deployed to the cluster.

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

## Isolate Kubernetes nodes

[Kubernetes Nodes](https://kubernetes.io/docs/concepts/architecture/nodes/) must be on a separate network and should not be exposed directly to public networks.
If possible, you should even avoid direct connections to the general corporate network.
This is only possible if Kubernetes control and data traffic are isolated.
Otherwise, both flow through the same pipe, and open access to the data plane implies open access to the control plane.
Ideally, Nodes should be configured with an ingress controller, set to only allow connections from the master node on the specified port through the network access control list.

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

## Notes

This document is not finalized.
