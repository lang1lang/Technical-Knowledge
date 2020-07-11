Operators extend two key Kubernetes concepts: *resources* and *controllers*. The Kubernetes API includes a mechanism, the CRD, for defining new resources. This chapter examines the Kubernetes objects Operators build on to add new capabilities to a cluster. It will help you understand how Operators fit into the Kubernetes architecture and why it is valuable to make an application a Kubernetes native.

# Standard Scaling: The ReplicaSet Resource

Looking at a standard resource, the ReplicaSet, gives a sense of how resources comprise the application management database at the heart of Kubernetes. Like any other resource in the Kubernetes API, the ReplicaSet is a collection of API objects. The ReplicaSet primarily collects pod objects forming a list of the running replicas of an application. The specification of another object type defines the number of those replicas that should be maintained on the cluster. A third object spec points to a template for creating new pods when there are fewer running than desired. There are more objects collected in a ReplicaSet, but these three types define the basic state of a scalable set of pods running on the cluster. Here, we can see these three key pieces for the staticweb ReplicaSet from Chapter 1 (the Selector, Replicas, and Pod Template fields):

```
$ kubectl describe replicaset/staticweb-69ccd6d6c
Name: staticweb-69ccd6d6c
Namespace: default
Selector: pod-template-hash=69ccd6d6c,run=staticweb
Labels: pod-template-hash=69ccd6d6c
 run=staticweb
Controlled By: Deployment/staticweb
Replicas: 1 current / 1 desired
Pods Status: 1 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
 Labels: pod-template-hash=69ccd6d6c
 run=staticweb
 Containers:
 staticweb:
 Image: nginx
```

A standard Kubernetes control plane component, the ReplicaSet controller, manages ReplicaSets and the pods belonging to them. The ReplicaSet controller creates ReplicaSets and continually monitors them. When the count of running pods doesn’t match the desired number in the Replicas field, the ReplicaSet controller starts or stops pods to make the actual state match the desired state.

The actions the ReplicaSet controller takes are intentionally general and application agnostic. It starts new replicas according to the pod template, or deletes excess pods. It does not, should not, and truly cannot know the particulars of startup and shutdown sequences for every application that might run on a Kubernetes cluster.

An Operator is the application-specific combination of CRs and a custom controller that does know all the details about *starting*, *scaling*, *recovering*, and *managing* its application. The Operator’s *operand* is what we call the application, service, or whatever resources an Operator manages.

# Custom Resources

CRs, as extensions of the Kubernetes API, contain one or more fields, like a native resource, but are not part of a default Kubernetes deployment. CRs hold structured data, and the API server provides a mechanism for reading and setting their fields as you would those in a native resource, by using kubectl or another API client. Users define a CR on a running cluster by providing a CR definition. A CRD is akin to a schema for a CR, defining *the CR’s fields* and the types of values those fields contain.























