An Operator is a way to package, run, and maintain a Kubernetes application. A Kubernetes application is not only deployed on Kubernetes, it is designed to use and to operate in concert with Kubernetes facilities and tools.

An Operator builds on Kubernetes abstractions to automate the entire lifecycle of the software it manages. Because they extend Kubernetes, Operators provide application-specific automation in terms familiar to a large and growing community. For application programmers, Operators make it easier to deploy and run the foundation services on which their apps depend. For infrastructure engineers and vendors, Operators provide a consistent way to distribute software on Kubernetes clusters and reduce support burdens by identifying and correcting application problems before the pager beeps.

Before we begin to describe how Operators do these jobs, let’s define a few Kubernetes terms to provide context and a shared language to describe Operator concepts and components.

# How Kubernetes Works

Kubernetes automates the lifecycle of a stateless application, such as a static web server. Without state, any instances of an application are interchangeable. This simple web server retrieves files and sends them on to a visitor’s browser. Because the server is not tracking state or storing input or data of any kind, when one server instance fails, Kubernetes can replace it with another. Kubernetes refers to these instances, each a copy of an application running on the cluster, as *replicas*.

A Kubernetes cluster is a collection of computers, called nodes. All cluster work runs on one, some, or all of a cluster’s nodes. The basic unit of work, and of replication, is the *pod*. A pod is a group of one or more Linux containers with common resources like networking, storage, and access to shared memory

At a high level, a Kubernetes cluster can be divided into two planes. The *control plane* is, in simple terms, Kubernetes itself. A collection of pods comprises the control plane and implements the Kubernetes application programming interface (API) and cluster orchestration logic.

The *application plane*, or data plane, is everything else. It is the group of nodes where application pods run. One or more nodes are usually dedicated to running applications, while one or more nodes are often sequestered to run only control plane pods. As with application pods, multiple replicas of control plane components can run on multiple controller nodes to provide redundancy.

The controllers of the control plane implement control loops that repeatedly compare the desired state of the cluster to its actual state. When the two diverge, a controller takes action to make them match. Operators extend this behavior. The schematic in Figure 1-1 shows the major control plane components, with worker nodes running application workloads.

While a strict division between the control and application planes is a convenient mental model and a common way to deploy a Kubernetes cluster to segregate workloads, the control plane components are a collection of pods running on nodes, like any other application. In small clusters, control plane components are often sharing the same node or two with application workloads.

The conceptual model of a cordoned control plane isn't quite so tidy, either. The kubelet agent running on every node is part of the control plane, for example. Likewise, an Operator is a type of controller, usually thought of as a control plane component. Operators can blur this distinct border between planes, however. **Treating the control and application planes as isolated domains is a helpful simplifying abstraction, not an absolute truth.**

# Example: Stateless Web Server

Since you haven't set up a cluster yet, the examples in this chapter are more like terminal excerpt “screenshots” that show what basic interactions between Kubernetes and an application look like. You are not expected to execute these commands as you are those throughout the rest of the book. In this first example, Kubernetes manages a relatively simple application and no Operators are involved.

Consider a cluster running a single replica of a stateless, static web server:

```
$ kubectl get pods
NAME READY STATUS RESTARTS AGE
staticweb-69ccd6d6c-9mr8l 1/1 Running 0 23s
```

After declaring there should be three replicas, the cluster’s actual state differs from the desired state, and Kubernetes starts two new instances of the web server to reconcile the two, scaling the web server deployment:


```
$ kubectl scale deployment staticweb --replicas=3
$ kubectl get pods
NAME READY STATUS RESTARTS AGE
staticweb-69ccd6d6c-4tdhk 1/1 Running 0 6s
staticweb-69ccd6d6c-9mr8l 1/1 Running 0 100s
staticweb-69ccd6d6c-m9qc7 1/1 Running 0 6s
```

Deleting one of the web server pods triggers work in the control plane to restore the desired state of three replicas. Kubernetes starts a new pod to replace the deleted one. In this excerpt, the replacement pod shows a STATUS of ContainerCreating:


```
$ kubectl delete pod staticweb-69ccd6d6c-9mr8l
$ kubectl get pods
NAME READY STATUS RESTARTS AGE
staticweb-69ccd6d6c-4tdhk 1/1 Running 0 2m8s
staticweb-69ccd6d6c-bk27p 0/1 ContainerCreating 0 14s
staticweb-69ccd6d6c-m9qc7 1/1 Running 0 2m8s
```

This static site’s web server is interchangeable with any other replica, or with a new pod that replaces one of the replicas. It doesn’t store data or maintain state in any way. Kubernetes doesn’t need to make any special arrangements to replace a failed pod, or to scale the application by adding or removing replicas of the server.

# Stateful Is Hard

Most applications have state. They also have particulars of startup, component interdependence, and configuration. They often have their own notion of what “cluster” means. They need to reliably store critical and sometimes voluminous data. Those are just three of the dimensions in which real-world applications must maintain state. It would be ideal to manage these applications with uniform mechanisms while automating their complex storage, networking, and cluster connection requirements.

Kubernetes cannot know all about every stateful, complex, clustered application while also remaining general, adaptable, and simple. It aims instead to provide *a set of flexible abstractions*, covering **the basic application concepts of scheduling, replication, and failover automation**, while providing **a clean extension mechanism** for more advanced or application-specific operations. Kubernetes, on its own, does not and should not know the configuration values for, say, a PostgreSQL database cluster, with its arranged memberships and stateful, persistent storage.

# Operators Are Software SREs

Site Reliability Engineering (SRE) is a set of patterns and principles for running large systems. Originating at Google, SRE has had a pronounced influence on industry practice. Practitioners must interpret and apply SRE philosophy to particular circumstances, but a key tenet is automating systems administration by writing software to run your software. Teams freed from rote maintenance work have more time to create new features, fix bugs, and generally improve their products.

An Operator is like an automated Site Reliability Engineer for its application. It encodes in software the skills of an expert administrator. An Operator can manage a cluster of database servers, for example. It knows the details of configuring and managing its application, and it can install a database cluster of a declared software version and number of members. An Operator continues to monitor its application as it runs, and can back up data, recover from failures, and upgrade the application over time, automatically. Cluster users employ kubectl and other standard tools to work with Oper‐ators and the applications they manage, because Operators extend Kubernetes.

# How Operators Work

Operators work by extending the Kubernetes control plane and API. In its simplest form, an Operator adds an endpoint to the Kubernetes API, called a *custom resource* (CR), along with a control plane component that monitors and maintains resources of the new type. This Operator can then take action based on the resource's state. This is illustrated in Figure 1-2.

kb get 