# First deployment

Let's deploy our sample application (the one you forked earlier).

We only have one Kubernetes cluster, so we will be deploying our app to the same cluster in which we installed Argo CD. You can also easily attach other clusters to Argo CD and use it to deploy apps to to those.

In the UI, click on `+ NEW APP`.

Fill in the form as follows, using details pertaining to your fork of the sample app. Make sure to replace the placeholder `YOUR-GITHUB-ORG` with the proper value. Leave any fields not mentioned below at their default value.
```
GENERAL
Application Name: spring-sample-app
Project: default
SYNC POLICY: Automatic

SOURCE
Repository URL: https://github.com/<YOUR-GITHUB-ORG>/spring-sample-app-ops.git
Revision: HEAD
Path: overlays/dev

DESTINATION
Cluster: https://kubernetes.default.svc
Namespace: default
```

Notice that the repo we have provided is the gitops repo, not the app source code repo. The gitops repo contains a reference to the app image, which Argo CD assumes has been created already. If you look through the contents of the repo, you'll see it specifies all of the necessary information that Kubernetes needs for deployment. You'll notice also that we've chosen to lay out our gitops yaml using Kustomize, which has advantages for re-use and simplicity at scale, but Argo CD would support a simpler yaml file layout as well.

Note also that the cluster we specified as our destination (aliased as  "in-cluster" by Argo CD by default) refers to the same cluster into which Argo CD is installed. It is possible to attach other clusters to Argo CD and deploy to those as well. Since we only have one cluster, we will just use the "in-cluster" option.

Before hitting 'CREATE', click on 'EDIT AS YAML'. Notice that you can define an application declaratively in yaml and create the app without using the UI.

Click 'CANCEL' to exit the yaml editor and then click 'CREATE' to create the app in Argo CD.

Wait till you see the new app appear. If you don't see it within a few moments, refresh the Dashboard tab.



Congrats! You've created your first app. Let's explore what's happened...