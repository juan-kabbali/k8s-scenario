# 1. Architecture & Process

Let's say we're building a new infrastructure and platform from scratch. The platform we're trying to build and deploy consists of a few applications and databases. It has to run on a Kubernetes stack. We're expecting you to provide guidelines on what should be deployed in regard to the following requirements:

### We should have observability

First, I will scope observability as the set of tools and techniques that allow us to collect data from our systems, 
in this case our k8s cluster, in order to understand what is happening, what happened and potentially what is going to happen.

Therefore, I propose to implement prometheus as a monitoring and observability stack, which is a widely used, well 
documented and open source. It offers many useful built-in functionalities, such as time series data storage and a 
powerful query language for interacting with data extracted from exporters.

In addition, prometheus can be easily installed thanks to the k8s prometheus operator.

Finally, prometheus is surrounded by an observability environment that can include the alert manager to define the 
conditions that trigger alerts, and grafana to create dashboards and a variety of charts to represent the scraped metrics.

### Access to Kubernetes components must be secured. Some users have full access whereas some others may have limited access.

Because security is a huge topic I'll decompose it in multiple points:

**network**
* k8s cluster nodes should be created within one or multiple **private** virtual networks to avoid exposing endpoints 
outside the k8s perimeter.

* only expose strictly necessary applications through an ingress or load balancer, if the application must be used 
internally, use services instead of ingress.

* implement ingress and egress network policies to define which components can interact with each other.

* define a global network policy to deny all traffic by default.


**authentication**  
* implement a two-factor authentication to the cloud provider where our k8s cluster will be hosted.


**authorization**
* use role-based access control rules to define who can access which resources and what actions are authorized.
* group highly related resources by namespaces and create access control rules for those namespaces.


**k8s manifests** 
* scan periodically manifest files to detect sensitive data like passwords, api keys, tokens, etc ...
  there are several tools that could help us and can be integrated on CI pipelines such as [detect-secrets](https://github.com/Yelp/detect-secrets), 
  [checkov](https://github.com/bridgecrewio/checkov) or even [gitguardian](https://github.com/GitGuardian)

* use k8s secrets objects to store _securely_ sensitive data. However, in an ideal scenario we could implement plugins that
  integrate external secrets managers or vaults with k8s, for example with [external-secrets](https://github.com/external-secrets/external-secrets)
  that will allow kubernetes to retrieve sensitive data directly from gcp secret manager, azure key-vault, hashicorp vault, etc ...

* implement a tool such a [popeye](https://popeyecli.io/) to scan our manifests in order to find misconfigurations that could leverage to security vulnerabilities or performance issues,
  for example, not defining pod probes, reference non-existent configmaps or not defining pod's resources limits.

### Our Kubernetes cluster should be fault tolerant, in the sense that a network outage in one cloud zone / region should not impact much our applications' up-time

If we need a high availability k8s cluster that is up event network outages, the answer is to implement a **multi-region cluster**. So 
when incidences happen on a region ,there'll be always the secondary region to respond the requests and running apps won't be impacted.


### Our Kubernetes cluster nodes should not have a public IP address. All incoming traffic should be handled by a Load-balancer

Deploy the k8s cluster inside a virtual private network with a set of private ips, then only expose our apps through 
services ( only visible inside the cluster ), and what need to be public, create an ingress load balancer to link incoming 
request from outside to the internal services.

### Some of our applications should be deployed in a separate subnet

If some applications must be deployed in separate subnets, we can use node affinity or node anti-affinity to define the 
nodes where our pods will be created. 

In this case, we can create for a specific app the affinity to match the nodes that are placed in the required subnet. 


### Developers' desktop environment should stick as close as possible to production

Environments homogeneity could be achieved by:

* Automate as much as possible the infrastructure with techs as terraform or pulumi.
* Prefer GitOps or CD strategies over manual modifications.
* Define a git branching model to attach a specific branch to an environment.

### We must have a CI/CD
### We need an efficient backup strategy for data related services

### How would you handle application configuration ?

Apps should be parameterizable outside themselves, this could be accomplished by using configMaps or env vars that are passed 
to the pods at creation time.

### What processes and tools do you plan to use in order to have a proper "Infrastructure As Code" approach

First, take the choice of what tool or tools could be used. Terraform is a well known tool that can be easily integrated with 
most cloud providers.

Then, define a centralized backend ( mechanism to store the current state of our cloud resources ) for allowing devs to 
interact with the same state.

Implement pipelines to handle CI CD actions, for example:

For CI:
* run a Terraform format to validate HCL syntax standard
* run a Terraform validate to validate code syntax
* run a Terraform init to validate all module dependencies and backend initialization

For CD:
* run a Terraform plan to know the delta
* run a Terraform apply to deploy the changes

### Would you recommend a full cloud approach ? Hybrid deployment ?

### How would you deploy stateful applications such as databases ? Would you have them within Kubernetes and if so how would you manage them ?

Absolutely yes. K8s offers a wide set of components to deploy stateful applications where data is critical and must be 
persisted like databases.

With ``StatefulSets`` we can deploy a set of pods where each one of them is attached to a persistent volume. So if a pod 
goes down, a new one will be created and attached to the persistent volume which stores the data.  


> Please provide information for each of the above points. Once again, please just state how you would achieve these things.

### Bonus Question

Imagine that one of your colleagues accidentally applied all the “production” config-maps to the “beta” environment.
### What impacts could this have ?

It will depend on how the apps are consuming the ConfigMaps values, normally ConfigMaps are consumed on start or restart pods, 
so it couldn't have an immediate impact, but when pods are created or restarted, they will be pointing to wrong 
configuration values and the beta env is going to be affected, because configMaps contains the required data to run apps 
in a target environment.

### How would you
#### Get things running properly again ?

* If we have a good CI/CD or GitOps strategy, we can run the deployment pipeline targeting the commit where with the right ConfigMaps
* In the case where the apps are affected, we can run a rollback to a healthy revision of our deployments.

#### Find what has been impacted ?

* The first reflex would be to go to grafana's dashboards and alert manager and check if any alerts were triggered.
* We can also check pod logs and look for container errors.
* We can check k8s events and look for k8s error events.
* List pods and check the status of replicas, for example if a replica defines 5 pods, so it must by 5/5 running. 

#### Prevent a re-occurrence of the issue ?

* Limit manual changes.
* Deploy only with pipelines, and important steps like apply changes must be manually approved.
* Take a 5 minutes pause before deploying xD 
