In the first section of this chapter we outline the requirements for running the examples in this book, and offer advice on how to establish access to a Kubernetes cluster that satisfies those requirements. In the second section, you’ll use that cluster to investigate what Operators do by installing and using one.

By the end, you’ll have a Kubernetes cluster to use as an Operator test bed, and you’ll know how to deploy an existing Operator on it from a set of manifests. You’ll also have seen an Operator managing its application’s specific internal state in the face of changes and failures, informing your understanding of the Operator architecture and build tools presented in succeeding chapters.

# Setting Up an Operator Lab

To build, test, and run Operators in the following chapters, you’ll need cluster admin access to a cluster running Kubernetes version v1.11.0 or later. If you’ve already met these requirements, you can skip ahead to the next section. In this section we offer general advice to readers who need to set up a Kubernetes cluster, or who need a local environment for Operator development and testing.

# Cluster Version Requirements

We’ve tested the examples in this book with Kubernetes releases v1.11 up to v1.16. We will state when any feature or action we examine requires a release later than v1.11.

## Control plane extensibility

Kubernetes version 1.2 introduced the API extension mechanism known as the CRD in elemental form as the *third party resource* (TPR). Since then, the components Operators build on have multiplied and matured, as illustrated in Figure 2-1. CRDs were formalized with the Kubernetes version 1.7 release.

