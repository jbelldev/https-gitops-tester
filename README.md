
# Table of Contents

1.  [Overview](#orgd792226)
    1.  [Purpose](#org3d6d948)
2.  [Installation](#org21fd45b)
3.  [Uninstalling](#orgaa34586)



<a id="orgd792226"></a>

# Overview

<span class="underline">https-gitops-tester</span> is a sample project for showcasing some standard usage of
ArgoCD in an OpenShift cluster. It maintains resources that will build a basic
httpd application based on resources managed in a GitOps style.

This project can be deployed by building an (ArgoCD) Application pointing to the
[src](./src) directory. One such Application is deployed at the root of the repo in
[application.yaml](./application.yaml); assuming the cluster has the baseline installation of
OpenShift GitOps, it can deploy the entirety of this project using
`oc apply -f ./application.yaml`.


<a id="org3d6d948"></a>

## Purpose

This is less of a sample project for httpd or GitOps so much as a useful
test-case for ArgoCD in OpenShift. The project is large, customizable, and
capable, and it can be useful to have a baseline to work against just to
establish expectations with a project slightly more complex than the usual
tutorial-level basics.


<a id="org21fd45b"></a>

# Installation

Installing and using <span class="underline">https-gitops-tester</span> is as cloning this repo, and adding the
Application manifest to the cluster:

    oc apply -f ./application.yaml

The default Application manifest includes the label needed for OpenShift GitOps&rsquo;
default ArgoCD instance to create and label (but not maintain) the namespace it
creates for this project, `https-gitops-tester`.

**The App will then need to synced**. Unfortunately, for a newly created namespace,
this will likely need to be done via the ArgoCD web console, or the `argocd`
binary (if installed).

Once synced, ArgoCD will create the destination namespace and deploy the
resources for the sample app to it. To finish installation, the image for the
Deployment must be created by the initial Build for the BuildConfig; this
resource can be watched using `oc wait`:

    oc wait --for=condition=Complete build/initial-https-builder -n https-gitops-tester

Once this build completes, the Deployment will finish, and the Route can be
queried to be served the site as defined by the [ConfigMap](src/configmap.yaml).


<a id="orgaa34586"></a>

# Uninstalling

Uninstalling from a cluster is just as simple, since ArgoCD manages most of the
manifests. From the web console, just deleting this Application with a cascading
deletion will handle most of the work on its own; from the CLI, the user will
need to first set the App to have a cascading deletion via finalizer:

    oc patch -f application.yaml -p '{"metadata": {"finalizers": ["resources-finalizer.argocd.argoproj.io"]}}' --type merge

Now, deleting the Application will result in *most* of the manifests being removed
as well:

    oc delete -f application.yaml

The namespace is the exception to this, since it isn&rsquo;t actually *managed* by
ArgoCD, it&rsquo;s just labeled by it. Completing the uninstall will involve deleting
the namespace as well:

    oc delete project https-gitops-tester

This will complete the removal of the project from the cluster. A fresh install
is possible from here, if needed for practice or testing.

