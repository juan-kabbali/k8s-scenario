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
