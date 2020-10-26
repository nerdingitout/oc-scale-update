# oc-scale-update
Scale, Update and Rollback your application deployment
## Content
- Prerequisites
- Overview
- Create Application Using S2I
- Scale Applications Using Replicas
- Update Application
- Roll back Applicatoin
- Summary

## Prerequisites
For this tutorial you will need:
 - Red Hat OpenShift Cluster 4.3 on IBM Cloud.
- oc CLI (can be downloaded from this <a href="https://mirror.openshift.com/pub/openshift-v4/clients/oc/4.3/">link</a>.
## Overview
Scaling deployment is increasing the number of Pods/Instances of a given deployment.<br>
In this tutorial, you will learn how to scale and how to update your deployment (move to a next version of the application) and roll back application if needed.
## Create Application Using S2I
1- Create a new project using the following command.<br>
```
oc new-project guestbook-project
```
2- Create a new deployment resource using the ibmcom/guestbook:v1 docker image in the project we just created.<br>
```
oc new-app myguestbook1 --image=ibmcom/guestbook:v1
```
3- This deployment creates the corresponding Pod that's in running state. Use the following command to see the list of pods in your namespace<br>
```
oc get pods
```
4- Expose the deployment on port 3000<br>
```
oc expose deployment myguestbook --type="NodePort" --port=3000
```
To view the service we just exposed. Use the following command.<br>
```
oc get service
```
Note that the service inside the pod is accessible using the <Node IP>:<NodePort>, but in case of OpenShift on IBM Cloud the NodeIP is not publicly accessible. One can use the built-in kubernetes terminal available in IBM Cloud which spawns a kubernetes shell which is part of the OpenShift cluster network and NodeID:NodePort is accessible from that shell.<br>
 5- To make the exposed service publicly accessible, create a public router with the following command.<br>
 ```
 DO THAT WITH THE WEB CONSOLE, AND DESCRIBE DETAILS
 ```
## Scale Applications Using Replicas
## Update Application
## Roll back Applicatoin
## Summary
