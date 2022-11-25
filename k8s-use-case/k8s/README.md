# k8s app components

In order to deploy the node js app, a set of kubernetes objects have been defined. the following is a broad description 
of the functionality of each one of them :

### namespace
a namespace called ``app`` to isolate our k8s resources

### configmap
a group of configuration values to pass to the node js app, for this case, only the ``PORT`` value is passed

### deployment
defines our pod specifications and creates a ReplicaSet to ensure that the number of replicas are healthy and running, 
for this case we use 3 replicas.

### service
exposes at a cluster level the node app through the port 80.

### ingress
expose the app externally through a load balancer, linking the outside traffic to the internal service.

### kustomization file
is used to put all components together and make easier the apply process, is can be also used to templatize manifests and 
avoid duplication. 

```kubectl apply -k k8s-use-case/k8s```

**the node js app is publicly exposed through the static ip [34.160.238.114](http://34.160.238.114/)**

### CI/CD

I would like to implement a CI / CD basics actions using [kubectl for gke](https://github.com/marketplace/actions/kubectl-google-cloud-gke-cluster)
but for the connection between the GitHub agent and GCP we need to create a service account, and I haven't permissions.

Anyway, I'll list the possible CI/CD actions that could be potentially implemented:

* CI
  * run a ``yamllint``
  * run a ``kustomization build``
  * run a ``detect-secrets`` to the previous build output
  
* CD
  * run a ``kubctl diff`` pointing to the kustomization file to get the entire stack difference instead of the diff of a single resource
  * run a ``kubectl apply -k k8s-use-case/k8s`` with manual approbation
