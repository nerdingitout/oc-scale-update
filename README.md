# Update and Rollback Deployment with Red Hat OpenShift
Scale, Update and Rollback your application deployment
## Introduction
Scaling deployment is increasing the number of Pods/Instances of a given deployment.<br>
In this tutorial, you will learn how to scale and how to update your deployment (move to a next version of the application) and roll back application if needed.
## Prerequisites
For this tutorial you will need:
 - Red Hat OpenShift Cluster 4.3 on IBM Cloud.
- oc CLI (can be downloaded from this <a href="https://mirror.openshift.com/pub/openshift-v4/clients/oc/4.3/">link</a> or you can use it at <a href="http://shell.cloud.ibm.com/">http://shell.cloud.ibm.com/.
## Estimated Time
It will take you around 30 minutes to complete this tutorial.

## Steps
- [Create Application Using S2I](https://github.com/nerdingitout/oc-scale-update#create-application-using-s2i)
- [Scale Applications Using Replicas](https://github.com/nerdingitout/oc-scale-update#scale-applications-using-replicas)
- [Update Application](https://github.com/nerdingitout/oc-scale-update#update-application)
- [Roll back Applicatoin](https://github.com/nerdingitout/oc-scale-update#roll-back-applicatoin)

## Create Application Using S2I
1- Create a new project using the following command.<br>
```
oc create namespace guestbook-project
```
2- Create a new deployment resource using the ibmcom/guestbook:v1 docker image in the project we just created.<br>
```
oc create deployment myguestbook --image=ibmcom/guestbook:v1
```
3- This deployment creates the corresponding Pod that's in running state. Use the following command to see the list of pods in your namespace<br>
```
oc get pods
```
![get pods](https://user-images.githubusercontent.com/36239840/97184468-602e4b00-17b8-11eb-8bba-9e8a8d396765.JPG)<br>

4- Expose the deployment on port 3000<br>
```
oc expose deployment myguestbook --type="NodePort" --port=3000
```
To view the service we just exposed. Use the following command.<br>
```
oc get service
```
![get service](https://user-images.githubusercontent.com/36239840/97184649-94a20700-17b8-11eb-93a3-15219a6b9845.JPG)

Note that the service inside the pod is accessible using the <Node IP>:<NodePort>, but in case of OpenShift on IBM Cloud the NodeIP is not publicly accessible. One can use the built-in kubernetes terminal available in IBM Cloud which spawns a kubernetes shell which is part of the OpenShift cluster network and NodeID:NodePort is accessible from that shell.<br>
 5- To make the exposed service publicly accessible, you will need to create a public router. First, go to <b>Networking &#8594; Routes</b> from the Administrator Perspective on the web console, then click 'Create Route'.<br>
 Fill in the information as follows:<br><br>
 <b>Name:</b>myguestbook<br>
 <b>Service:</b>myguestbook<br>
 <b>Target Port:</b>3000&#8594;3000(TCP)<br><br>
 Then click 'Create', you can leave the rest of the fields empty.
![create route](https://user-images.githubusercontent.com/36239840/97185180-3164a480-17b9-11eb-9fd3-1da5b8864c43.JPG)
Once created, you will be redirected to 'Route Overview' where you can access the external route from the URL under Location as shown in the screenshot. The Route has been created successfully, and it provides a publicly accessible endpoint URL using which we can access our guestbook application.<br>
If you click on the URL, you will be redirected to a page that looks like the following.<br>
![guestbook app](https://user-images.githubusercontent.com/36239840/97185901-15adce00-17ba-11eb-88a0-ae727879429c.JPG)
<br>Now that you have successfully deployed the application using S2I, you will be learning how to scale and rollback your application
## Scale Applications Using Replicas
1- Increase the capacity from a single running instance of guestbook up to 5 instances.<br>
```
oc scale --replicas=5 deployment/myguestbook
```
2- Check the status of the deployment<br>
```
oc rollout status deployment myguestbook
```
3- Verify that you have 5 pods running post the ```oc scale``` command.<br>
```
oc get pods -n guestbook-project
```
ADD SCREENSHOT HERE!!
## Update Application
1- Update your deployment using the v2 image.<br>
```
oc set image deployment/myguestbook guestbook=ibmcom/guestbook:v2
```
2- Check the status of the rollout.<br>
```
oc rollout status deployment/myguestbook
```
3- Get a ist of pods (newly launched) as part of moving to the v2 image.<br>
```
oc get pods -n guestbook-project
```
4- Copy the name of one of the new pods and use ```oc describe pod``` to confirm that the pod is using the v2 image like the screenshot shown below.<br>
```
oc describe pods/myguestbook........
```

## Roll back Applicatoin
## Summary
