In the first section of this chapter we outline the requirements for running the examples in this book, and offer advice on how to establish access to a Kubernetes cluster that satisfies those requirements. In the second section, you’ll use that cluster to investigate what Operators do by installing and using one.

By the end, you’ll have a Kubernetes cluster to use as an Operator test bed, and you’ll know how to deploy an existing Operator on it from a set of manifests. You’ll also have seen an Operator managing its application’s specific internal state in the face of changes and failures, informing your understanding of the Operator architecture and build tools presented in succeeding chapters.

# Setting Up an Operator Lab

To build, test, and run Operators in the following chapters, you’ll need cluster admin access to a cluster running Kubernetes version v1.11.0 or later. If you’ve already met these requirements, you can skip ahead to the next section. In this section we offer general advice to readers who need to set up a Kubernetes cluster, or who need a local environment for Operator development and testing.

# Cluster Version Requirements

We’ve tested the examples in this book with Kubernetes releases v1.11 up to v1.16. We will state when any feature or action we examine requires a release later than v1.11.

## Control plane extensibility

Kubernetes version 1.2 introduced the API extension mechanism known as the CRD in elemental form as the *third party resource* (TPR). Since then, the components Operators build on have multiplied and matured, as illustrated in Figure 2-1. CRDs were formalized with the Kubernetes version 1.7 release.

![Figure 2-1. Extensibility features per Kubernetes release.png](https://github.com/lang1lang/Technical-Knowledge/blob/master/Kubernetes%20Operators-%E8%87%AA%E5%8A%A8%E5%8C%96%E5%AE%B9%E5%99%A8%E7%BC%96%E6%8E%92%E5%B9%B3%E5%8F%B0/images/Figure%202-1.%20Extensibility%20features%20per%20Kubernetes%20release.png?raw=true)

As you saw in Chapter 1, a CRD is the definition of a new, site-specific resource (or API endpoint) in the Kubernetes API of a particular cluster. CRDs are one of two essential building blocks for the most basic description of the Operator pattern: **a custom controller managing CRs**.

# Authorization Requirements

Since Operators extend Kubernetes itself, you’ll need privileged, cluster-wide access to a Kubernetes cluster to deploy them, such as the common cluster-admin cluster role.

> Less privileged users can use the services and applications that Operators manage—the “operands.”

While you should configure more granular Kubernetes Role-Based Access Control (RBAC) for production scenarios, having complete control of your cluster means you’ll be able to deploy CRDs and Operators immediately. You’ll also have the power to declare more detailed RBAC as you develop the roles, service accounts, and bindings for your Operators and the applications they manage.

You can ask the Kubernetes API about the cluster-admin role to see if it exists on your cluster. The following shell excerpt shows how to get a summary of the role with the kubectl’s describe subcommand:

```
$ kubectl describe clusterrole cluster-admin
Name: cluster-admin 
Labels: kubernetes.io/bootstrapping=rbac-defaults
PolicyRule:
 Resources Non-Resource URLs Resource Names Verbs
 --------- ----------------- -------------- -----
 *.* [] [] [*]
 [*] [] [*]
```

> The RBAC cluster-admin ClusterRole: anything goes.

# Standard Tools and Techniques

Operators aim to make the complex applications they manage first-class citizens of the Kubernetes API. We show what that means in the following chapters’ examples. At this stage, it means that a recent version of the command-line Kubernetes API tool, kubectl, is the only requirement for deploying and interacting with basic Operators on your cluster.

Readers who need to install or update kubectl should consult the current documentation.

> Users of the Red Hat OpenShift Kubernetes distribution (described below) may optionally (and interchangeably) use the oc OpenShift API utility in place of kubectl.

# Suggested Cluster Configurations

There are many ways to run a Kubernetes cluster where you can deploy Operators. As mentioned previously, if you are already running a recent Kubernetes version, you can skip past this advice and on to “Running a Simple Operator” on page 13. If you aren’t, we have tested the Kubernetes packagings or distributions described in this section enough to expect they will support the exercises in this book.

## Minikube

Minikube v1.5.2 deploys Kubernetes v1.16.2. It runs a single-node Kubernetes cluster in a virtual machine (VM) on your local system’s hypervisor. By default, Minikube expects to use VirtualBox because of its wide availability, but with a few extra steps it can also use your platform’s native hypervisor, like KVM on Linux, Hyper-V on Windows, or HyperKit and Hypervisor.framework on macOS. We avoid detailed installation instructions here, because they are better left to the Minikube documentation. We have tested the examples in this book most thoroughly with Minikube, and for reasons of convenience and cost we are recommending that you start your Operator experiments with a local environment like it, CodeReady Containers (see the next section), or with Kubernetes in Docker (kind).

## Red Hat OpenShift

OpenShift is Red Hat’s distribution of Kubernetes. Anything you can do on Kubernetes, you can do on OpenShift of an equivalent core version. (There are also OpenShift-specific features built atop Kubernetes, but those are beyond the scope of this book.) OpenShift version 4 provides a full-featured Kubernetes distribution that is itself designed, delivered, and managed using Operators. It’s a **“self-hosted” Kubernetes**, capable of performing in-place platform upgrades without incurring downtime for hosted workloads. OpenShift includes Operator Lifecycle Manager, described in Chapter 4, and a graphical interface to the Operator Catalog distribution mechanism out of the box.

You can deploy a fully fledged OpenShift v4 cluster on Amazon Web Services (AWS), Microsoft Azure, or Google Cloud Platform with a free trial license by visiting Red Hat’s *https://try.openshift.com*.

> To run OpenShift on your laptop, take a look at Minikube’s equivalent, Red Hat CodeReady Containers.

> OpenShift Learning Portal
> The OpenShift learning portal offers guided lessons, including access to a cluster with all the necessary privileges for installing, deploying, and managing Operators. Scenarios are available in your web browser, making it easy to keep learning beyond the examples in this book. An OpenShift cluster spins up for each session, and you’re given command-line and web GUI access to it.
> To check it out, visit https://learn.openshift.com and select the “Building Operators on OpenShift” group of topics.

# Checking Your Cluster Version

Verify that your cluster is running Kubernetes version v1.11 or later by running kubectl version. This command will return one API version string for your kubectl binary and a second version string for the cluster to which it is connecting:

```
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.2", GitCommit:"c97fe5036ef3df2967d086711e6c0c405941e14b", GitTreeState:"clean", BuildDate:"2019-10-15T19:18:23Z", GoVersion:"go1.12.10", Compiler:"gc",
Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.2",
GitCommit:"c97fe5036ef3df2967d086711e6c0c405941e14b", GitTreeState:"clean",
BuildDate:"2019-10-15T19:09:08Z", GoVersion:"go1.12.10", Compiler:"gc",
Platform:"linux/amd64"}
```

In the preceding output, both client and server are running Kubernetes version 1.16.2. While a kubectl client up to one release behind the server should work, for simplicity, you should make sure your client and server minor versions match. If you have v1.11 or later, you’re ready to start experimenting with Operators.

# Running a Simple Operator

Once you’ve verified that you have privileged access to a Kubernetes cluster of a compatible version, you’re ready to deploy an Operator and see what Operators can do. You’ll see the skeleton of this same procedure again later, when you deploy and test the Operator you build. The etcd Operator’s straightforward automation of recovery and upgrades shows the principles and goals of Kubernetes Operators in action.

## A Common Starting Point

etcd is a distributed key-value store with roots at CoreOS, now under the auspices of the Cloud Native Computing Foundation. It is the underlying data store at the core of Kubernetes, and a key piece of several distributed applications. etcd provides reliable storage by implementing a protocol called Raft that guarantees consensus among a quorum of members.

The etcd Operator often serves as a kind of “Hello World” example of the value and mechanics of the Operator pattern, and we follow that tradition here. We return to it because the most basic use of etcd is not difficult to illustrate, but etcd cluster setup and administration require exactly the kind of application-specific know-how you can bake into an Operator. To use etcd, you *put* keys and values in, and *get* them back out by name. Creating a reliable etcd cluster of the minimum three or more nodes requires configuration of endpoints, auth, and other concerns usually left to an etcd expert (or their collection of custom shell scripts). Keeping etcd running and upgraded over time requires continued administration. The etcd Operator knows how to do all of this.

In the sections that follow, you’ll deploy the etcd Operator, then have it create an etcd cluster according to your specifications. You will have the Operator recover from failures and perform a version upgrade while the etcd API continues to service read and write requests, showing how an Operator automates the lifecycle of a piece of foundation software.

> You can follow this example on a running OpenShift cluster without doing any setup at the OpenShift learning portal.

# Fetching the etcd Operator Manifests

This book provides an accompanying Git repository for each chapter’s example code. Grab the *chapters* repo and change into Chapter 3’s examples directory, as shown here:

```
$ git clone https://github.com/kubernetes-operators-book/chapters.git
$ cd chapters/ch03
```

# CRs: Custom API Endpoints

As with nearly everything in Kubernetes, a YAML manifest describes a CRD. A CR is a named endpoint in the Kubernetes API. A CRD named *etcdclusters.etcd.database.coreos.com* represents the new type of endpoint.

## Creating a CRD

A CRD defines the types and values within an instance of a CR. This example defines a new *kind* of resource, the EtcdCluster.

Use cat, less, or your preferred pager to read the file named *etcd-operator-crd.yaml*. You’ll see something like the following, the YAML that specifies the EtcdCluster CRD:

```
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: etcdclusters.etcd.database.coreos.com
spec:
  group: etcd.database.coreos.com
  names:
    kind: EtcdCluster
    listKind: EtcdClusterList
    plural: etcdclusters
    shortNames:
    - etcdclus
    - etcd
    singular: etcdcluster
  scope: Namespaced
  versions:
  - name: v1beta2
    served: true
    storage: true
```

The CRD defines how the Kubernetes API should reference this new resource. The shortened nicknames that help you do a little less typing in kubectl are defined here, too.

Create the CRD on your cluster:

```
$ kubectl create -f etcd-operator-crd.yaml
```

A quick check shows the new CRD, *etcdclusters.etcd.database.coreos.com*:

```
$ kubectl get crd
NAME CREATED AT
etcdclusters.etcd.database.coreos.com 2019-11-15T02:50:14Z
```

> The CR’s group, version, and kind together form the fully qualified name of a Kubernetes resource type. That canonical name must be unique across a cluster. The CRD you created represents a resource in the *etcd.database.coreos.com* group, of version v1beta2 and kind EtcdCluster.

# Who Am I: Defining an Operator Service Account

In Chapter 3 we give an overview of Kubernetes authorization and define service accounts, roles, and other authorization concepts. For now, we just want to take a first look at basic declarations for a service account and the capabilities that account needs to run the etcd Operator.

The file *etcd-operator-sa.yaml* defines the service account:

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: etcd-operator-sa
```

Create the service account by using kubectl create:

```
$ kubectl create -f etcd-operator-sa.yaml
serviceaccount/etcd-operator-sa created
```

If you check the list of cluster service accounts, you’ll see that it appears:

```
$ kubectl get serviceaccounts
NAME SECRETS AGE
builder 2 2h
default 3 2h
deployer 2 2h
etcd-operator-sa 2 3s
[...]
```

## The role

The role governing the service account is defined in a file named *etcd-operator-role.yaml*. We’ll leave aside a detailed discussion of RBAC for later chapters and Appendix C, but the key items are fairly visible in the role manifest. We give the role a name that we’ll use to reference it from other places: etcd-operator-role. The YAML goes on to list the kinds of resources the role may use, and what it can do with them, that is, what verbs it can say:

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: etcd-operator-role
rules:
- apiGroups:
  - etcd.database.coreos.com
  resources:
  - etcdclusters
  - etcdbackups
  - etcdrestores
  verbs:
  - '*'
- apiGroups:
  - ""
  resources:
  - pods
  - services
  - endpoints
  - persistentvolumeclaims
  - events
  verbs:
  - '*'
- apiGroups:
  - apps
  resources:
  - deployments
  verbs:
  - '*'
- apiGroups:
  - ""
  resources:
  - secrets
  verbs:
  - get
```

As with the service account, create the role with kubectl:

```
$ kubectl create -f etcd-operator-role.yaml
role.rbac.authorization.k8s.io/etcd-operator-role created
```

## Role binding

The last bit of RBAC configuration, RoleBinding, assigns the role to the service account for the etcd Operator. It’s declared in the file *etcd-operator-rolebinding.yaml*:

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: etcd-operator-rolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: etcd-operator-role
subjects:
- kind: ServiceAccount
  name: etcd-operator-sa
  namespace: default
```

Notice the last line. If you’re on a brand-new OpenShift cluster, like that provided by CodeReady Containers, by default your kubectl or oc commands will run in the namespace *myproject*. If you’re on a similarly unconfigured Kubernetes cluster, your context’s default will usually be the namespace default. Wherever you are, the namespace value in this RoleBinding must match the namespace on the cluster where you are working.

Create the binding now:

```
$ kubectl create -f etcd-operator-rolebinding.yaml
rolebinding.rbac.authorization.k8s.io/etcd-operator-rolebinding created
```

# Deploying the etcd Operator

The Operator is a custom controller running in a pod, and it watches the EtcdCluster CR you defined earlier. The manifest file *etcd-operator-deployment.yaml* lays out the Operator pod’s specification, including the container image for the Operator you’re deploying. Notice that it does not define the spec for the etcd cluster. You’ll describe the desired etcd cluster to the deployed etcd Operator in a CR once the Operator is running:

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    name: etcdoperator
  name: etcd-operator
spec:
  replicas: 1
  selector:
    name: etcd-operator
  template:
   name: etcd-operator
   spec:
     containers:
     - name: etcd-operator
       image: quay.io/coreos/etcd-operator:v0.9.4
       command:
       - etcd-operator
       - --create-crd=false
       [...]
       imagePullPolicy: IfNotPresent
       serviceAccountName: etcd-operator-sa
```

The deployment provides labels and a name for your Operator. Some key items to note here are the container image to run in this deployment’s pods, etcd-operator:v0.9.4, and the service account the deployment’s resources should use to access the cluster’s Kubernetes API. The etcd-operator deployment uses the etcd-operator-sa service account created for it.

As usual, you can create this resource on the cluster from the manifest:

```
$ kubectl create -f etcd-operator-deployment.yaml
deployment.apps/etcd-operator created
$ kubectl get deployments
NAME DESIRED CURRENT UP-TO-DATE AVAILABLE AGE
etcd-operator 1 1 1 1 19s
```

The etcd Operator is itself a pod running in that deployment. Here you can see it starting up:

```
$ kubectl get pods
NAME READY STATUS RESTARTS AGE
etcd-operator-594fbd565f-4fm8k 0/1 ContainerCreating 0 4s
```

# Declaring an etcd Cluster

Earlier, you created a CRD defining a new kind of resource, an EtcdCluster. Now that you have an Operator watching EtcdCluster resources, you can declare an EtcdCluster with your desired state. To do so, provide the two spec elements the Operator recognizes: size, the number of etcd cluster members, and the version of etcd each of those members should run.

You can see the spec stanza in the file *etcd-cluster-cr.yaml*:

```
apiVersion: etcd.database.coreos.com/v1beta2
kind: EtcdCluster
metadata:
  name: example-etcd-cluster
spec:
  size: 3
  version: 3.1.10
```

This brief manifest declares a desired state of three cluster members, each running version 3.1.10 of the etcd server. Create this etcd cluster using the familiar kubectl syntax:

```
$ kubectl create -f etcd-cluster-cr.yaml
etcdcluster.etcd.database.coreos.com/example-etcd-cluster created
$ kubectl get pods -w
NAME READY STATUS RESTARTS AGE
etcd-operator-594fbd565f-4fm8k 1/1 Running 0 3m
example-etcd-cluster-95gqrthjbz 1/1 Running 2 38s
example-etcd-cluster-m9ftnsk572 1/1 Running 0 34s
example-etcd-cluster-pjqhm8d4qj 1/1 Running 0 31s
```

This example etcd cluster is a first-class citizen, an EtcdCluster in your cluster’s API. Since it’s an API resource, you can get the etcd cluster spec and status directly from Kubernetes. Try kubectl describe to report on the size, etcd version, and status of your etcd cluster, as shown here:

```
$ kubectl describe etcdcluster/example-etcd-cluster
Name: example-etcd-cluster
Namespace: default
API Version: etcd.database.coreos.com/v1beta2
Kind: EtcdCluster
[...]
Spec:
  Repository: quay.io/coreos/etcd
  Size: 3
  Version: 3.1.10
Status:
  Client Port: 2379
  Conditions:
    Last Transition Time: 2019-11-15T02:52:04Z
    Reason: Cluster available
    Status: True
    Type: Available
  Current Version: 3.1.10
  Members:
    Ready:
      example-etcd-cluster-6pq7qn82g2
      example-etcd-cluster-dbwt7kr8lw
      example-etcd-cluster-t85hs2hwzb
  Phase: Running
  Service Name: example-etcd-cluster-client
```

# Exercising etcd

You now have a running etcd cluster. The etcd Operator creates a Kubernetes *service* in the etcd cluster’s namespace. A service is an endpoint where clients can obtain access to a group of pods, even though the members of the group may change. A service by default has a DNS name visible in the cluster. The Operator constructs the name of the service used by clients of the etcd API by appending-client to the etcd cluster name defined in the CR. Here, the client service is named example-etcd-cluster-client, and it listens on the usual etcd client IP port, 2379. Kubectl can list the services associated with the etcd cluster:

```
$ kubectl get services --selector etcd_cluster=example-etcd-cluster
NAME TYPE CLUSTER-IP ... PORT(S) AGE
example-etcd-cluster ClusterIP None ... 2379/TCP,2380/TCP 21h
example-etcd-cluster-client ClusterIP 10.96.46.231 ... 2379/TCP 21h
```

> The other service created by the etcd Operator, example-etcd-cluster, is utilized by etcd cluster members rather than etcd API clients.

You can run the etcd client on the cluster and use it to connect to the client service and interact with the etcd API. The following command lands you in the shell of an etcd container:

```
$ kubectl run --rm -i --tty etcdctl --image quay.io/coreos/etcd \
 --restart=Never -- /bin/sh
```

From the etcd container’s shell, create and read a key-value pair in etcd with etcdctl’s put and get verbs:

```
$ export ETCDCTL_API=3
$ export ETCDCSVC=http://example-etcd-cluster-client:2379
$ etcdctl --endpoints $ETCDCSVC put foo bar
$ etcdctl --endpoints $ETCDCSVC get foo
foo
bar
```

Repeat these queries or run new put and get commands in an etcdctl shell after each of the changes you go on to make. You’ll see the continuing availability of the etcd API service as the etcd Operator grows the cluster, replaces members, and upgrades the version of etcd.

## Scaling the etcd Cluster

You can grow the etcd cluster by changing the declared size specification. Edit *etcd-cluster-cr.yaml* and change size from 3 to 4 etcd members. Apply the changes to the EtcdCluster CR:

```
$ kubectl apply -f etcd-cluster-cr.yaml
```

Checking the running pods shows the Operator adding a new etcd member to the etcd cluster:

```
$ kubectl get pods
NAME READY STATUS RESTARTS AGE
etcd-operator-594fbd565f-4fm8k 1/1 Running 1 16m
example-etcd-cluster-95gqrthjbz 1/1 Running 2 15m
example-etcd-cluster-m9ftnsk572 1/1 Running 0 15m
example-etcd-cluster-pjqhm8d4qj 1/1 Running 0 15m
example-etcd-cluster-w5l67llqq8 0/1 Init:0/1 0 3s
```

> You can also try kubectl edit etcdcluster/example-etcd-cluster to drop into an editor and make a live change to the cluster size.

## Failure and Automated Recovery

You saw the etcd Operator replace a failed member back in Chapter 1. Before you see it live, it’s worth reiterating the general steps you’d have to take to handle this manually. Unlike a stateless program, no etcd pod runs in a vacuum. Usually, a human etcd “operator” has to notice a member’s failure, execute a new copy, and provide it with configuration so it can join the etcd cluster with the remaining members. The etcd Operator understands etcd’s internal state and makes the recovery automatic.

### Recovering from a failed etcd member

Run a quick kubectl get pods -l app=etc to get a list of the pods in your etcd cluster. Pick one you don’t like the looks of, and tell Kubernetes to delete it:

```
$ kubectl delete pod example-etcd-cluster-95gqrthjbz
pod "example-etcd-cluster-95gqrthjbz" deleted
```

The Operator notices the difference between reality on the cluster and the desired state, and adds an etcd member to replace the one you deleted. You can see the new etcd cluster member in the PodInitializing state when retrieving the list of pods, as shown here:

```
$ kubectl get pods -w
NAME READY STATUS RESTARTS AGE
etcd-operator-594fbd565f-4fm8k 1/1 Running 1 18m
example-etcd-cluster-m9ftnsk572 1/1 Running 0 17m
example-etcd-cluster-pjqhm8d4qj 1/1 Running 0 17m
example-etcd-cluster-r6cb8g2qqw 0/1 PodInitializing 0 31s
```

The -w switch tells kubectl to “watch” the list of pods and to print updates on its standard output with every change to the list. You can stop the watch and return to your shell prompt with Ctrl-C.

You can check the Events to see the recovery actions logged in the example-etcd-cluster CR:

```
$ kubectl describe etcdcluster/example-etcd-cluster
[...]
Events:
 Normal Replacing Dead Member 4m etcd-operator-589c65bd9f-hpkc6
 The dead member example-etcd-cluster-95gqrthjbz is being replaced
 Normal Member Removed 4m etcd-operator-589c65bd9f-hpkc6
 Existing member example-etcd-cluster-95gqrthjbz removed from the cluster
[...]
```

Throughout the recovery process, if you fire up the etcd client pod again, you can make requests to the etcd cluster, including a check on its general health:

```
$ kubectl run --rm -i --tty etcdctl --image quay.io/coreos/etcd \
 --restart=Never -- /bin/sh
If you don't see a command prompt, try pressing enter.
$ etcdctl --endpoints http://example-etcd-cluster-client:2379 cluster-health
member 5ee0dd47065a4f55 is healthy: got healthy result ...
member 70baca4290889c4a is healthy: got healthy result ...
member 76cd6c58798a7a4b is healthy: got healthy result ...
cluster is healthy
$ exit
pod "etcdctl" deleted
```

The etcd Operator recovers from failures in its complex, stateful application the same way Kubernetes automates recoveries for stateless apps. That is conceptually simple but operationally powerful. Building on these concepts, Operators can perform more advanced tricks, like upgrading the software they manage. Automating upgrades can have a positive impact on security, just by making sure things stay up to date. When an Operator performs rolling upgrades of its application while maintaining service availability, it’s easier to keep software patched with the latest fixes.

## Upgrading etcd Clusters

If you happen to be an etcd user already, you may have noticed we specified an older version, 3.1.10. We contrived this so we could explore the etcd Operator’s upgrade skills.

### Upgrading the hard way

At this point, you have an etcd cluster running version 3.1.10. To upgrade to etcd 3.2.13, you need to perform a series of steps. Since this book is about Operators, and not etcd administration, we’ve condensed the process presented here, leaving aside networking and host-level concerns to outline the manual upgrade process. The steps to follow to upgrade manually are:

1. Check the version and health of each etcd node.
2. Create a snapshot of the cluster state for disaster recovery
3. Stop one etcd server. Replace the existing version with the v3.2.13 binary. Start the new version.
4. Repeat for each etcd cluster member—at least two more times in a three-member cluster.

For the gory details, see the etcd upgrade documentation.

### The easy way: Let the Operator do it

With a sense of the repetitive and error-prone process of a manual upgrade, it’s easier to see the power of encoding that etcd-specific knowledge in the etcd Operator. The Operator can manage the etcd version, and an upgrade becomes a matter of declaring a new desired version in an EtcdCluster resource.

### Triggering etcd upgrade

Get the version of the current etcd container image by querying some etcd-cluster pod, filtering the output to see just the version:

```
$ kubectl get pod example-etcd-cluster-795649v9kq -o yaml | grep "image:" | uniq
image: quay.io/coreos/etcd:v3.1.10
image: busybox:1.28.0-glibc
```

Or, since you added an EtcdCluster resource to the Kubernetes API, you can instead summarize the Operator’s picture of example-etcd-cluster directly by using kubectl describe as you did earlier:

```
$ kubectl describe etcdcluster/example-etcd-cluster
```

You’ll see the cluster is running etcd version 3.1.10, as specified in the file *etcd-cluster-cr.yaml* and the CR created from it.

Edit *etcd-cluster-cr.yaml* and change the version spec from 3.1.10 to 3.2.13. Then apply the new spec to the resource on the cluster:

```
$ kubectl apply -f etcd-cluster-cr.yaml
```

Use the describe command again and take a look at the current and target versions, as well as the member upgrade notices in the Events stanza:

```
$ kubectl describe etcdcluster/example-etcd-cluster
Name: example-etcd-cluster
Namespace: default
API Version: etcd.database.coreos.com/v1beta2
Kind: EtcdCluster
[...]
Status:
  Conditions:
    [...]
    Message: upgrading to 3.2.13
    Reason: Cluster upgrading
    Status: True
    Type: Upgrading
  Current Version: 3.1.10
  [...]
  Size: 3
  Target Version: 3.2.13
Events:
  Type Reason Age From ...
  ---- ------ --- ---- ---
  Normal Member Upgraded 3s etcd-operator-594fbd565f-4fm8k ...
  Normal Member Upgraded 5s etcd-operator-594fbd565f-4fm8k ...
```

### Upgrade the upgrade

With some kubectl tricks, you can make the same edit directly through the Kubernetes API. This time, let’s upgrade from 3.2.13 to the latest minor version of etcd available at the time of this writing, version 3.3.12:

```
$ kubectl patch etcdcluster example-etcd-cluster --type='json' \
 -p '[{"op": "replace", "path": "/spec/version", "value":3.3.12}]'
```

Remember you can always make this change in the etcd cluster’s CR manifest and then apply it with kubectl, as you did to trigger the first upgrade.

Consecutive kubectl describe etcdcluster/example-etcd-cluster commands

will show the transition from the old version to a target version until that becomes the current version, at which point you’ll see Current Version: 3.3.12. The Events section records each of those upgrades:

```
 Normal Member Upgraded 1m etcd-operator-594fbd565f-4fm8k
 Member example-etcd-cluster-pjqhm8d4qj upgraded from 3.1.10 to 3.2.23
 Normal Member Upgraded 27s etcd-operator-594fbd565f-4fm8k
 Member example-etcd-cluster-r6cb8g2qqw upgraded from 3.2.23 to 3.3.12
```

# Cleaning Up

Before proceeding, it will be helpful if you remove the resources you created and manipulated to experiment with the etcd Operator. As shown in the following shell excerpt, you can remove resources with the manifests used to create them. First, ensure your current working directory is *ch03* inside the *chapters* Git repository you cloned earlier (cd chapters/ch03):

```
$ kubectl delete -f etcd-operator-sa.yaml
$ kubectl delete -f etcd-operator-role.yaml
$ kubectl delete -f etcd-operator-rolebinding.yaml
$ kubectl delete -f etcd-operator-crd.yaml
$ kubectl delete -f etcd-operator-deployment.yaml
$ kubectl delete -f etcd-cluster-cr.yaml
serviceaccount "etcd-operator-sa" deleted
role.rbac.authorization.k8s.io "etcd-operator-role" deleted
rolebinding.rbac.authorization.k8s.io "etcd-operator-rolebinding" deleted
customresourcedefinition.apiextensions.k8s.io \
 "etcdclusters.etcd.database.coreos.com" deleted
deployment.apps "etcd-operator" deleted
etcdcluster.etcd.database.coreos.com "example-etcd-cluster" deleted
```

# Summary

We use the etcd API here with the etcdctl tool for the sake of simplicity, but an application uses etcd with the same API requests, storing, retrieving, and watching keys and ranges. The etcd Operator automates the etcd cluster part, making reliable key-value storage available to more applications.

Operators get considerably more complex, managing a variety of concerns, as you would expect from application-specific extensions. Nevertheless, most Operators follow the basic pattern discernable in the etcd Operator: **a CR specifies some desired state**, such as the version of an application, and **a custom controller watches the resource, maintaining the desired state on the cluster**.

You now have a Kubernetes cluster for working with Operators. You’ve seen how to deploy an Operator and triggered it to perform application-specific state reconciliation. Next, we’ll introduce the Kubernetes API elements on which Operators build before introducing the Operator Framework and SDK, the toolkit you’ll use to construct an Operator.