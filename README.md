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
![get pods](https://user-images.githubusercontent.com/36239840/97298061-5534f280-186c-11eb-9dbb-982f2f1c20e0.JPG)

4- Expose the deployment on port 3000<br>
```
oc expose deployment myguestbook --type="NodePort" --port=3000
```
To view the service we just exposed. Use the following command.<br>
```
oc get service
```
![get service](https://user-images.githubusercontent.com/36239840/97298080-5d8d2d80-186c-11eb-8574-e39b2cb48105.JPG)

Note that the service inside the pod is accessible using the <Node IP>:<NodePort>, but in case of OpenShift on IBM Cloud the NodeIP is not publicly accessible. One can use the built-in kubernetes terminal available in IBM Cloud which spawns a kubernetes shell which is part of the OpenShift cluster network and NodeID:NodePort is accessible from that shell.<br>
 5- To make the exposed service publicly accessible, you will need to create a public router. First, go to <b>Networking &#8594; Routes</b> from the Administrator Perspective on the web console, then click 'Create Route'.<br>
 Fill in the information as follows:<br><br>
 <b>Name:</b>myguestbook<br>
 <b>Service:</b>myguestbook<br>
 <b>Target Port:</b>3000&#8594;3000(TCP)<br><br>
 Then click 'Create', you can leave the rest of the fields empty.<br>
![create route](https://user-images.githubusercontent.com/36239840/97185180-3164a480-17b9-11eb-9fd3-1da5b8864c43.JPG)
Once created, you will be redirected to 'Route Overview' where you can access the external route from the URL under Location as shown in the screenshot. The Route has been created successfully, and it provides a publicly accessible endpoint URL using which we can access our guestbook application.<br>
If you click on the URL, you will be redirected to a page that looks like the following.<br>
![guestbook v1 app](https://user-images.githubusercontent.com/36239840/97298686-3edb6680-186d-11eb-8c0a-f6e7bc5ae9c4.JPG)
<br>Now that you have successfully deployed the application using S2I, you will be learning how to scale and rollback your application
## Scale Applications Using Replicas
In this section, you will be scaling applications by creating replicas of the pod you created in the first section. Having multiple replicas of a pod can help you ensure that your deployment has the available resources to handle increasing load on your application.<br>
1- Increase the capacity from a single running instance of guestbook up to 5 instances.<br>
```
oc scale --replicas=5 deployment/myguestbook
```
2- Check the status of the deployment<br>
```
oc rollout status deployment myguestbook
```
![rollout](https://user-images.githubusercontent.com/36239840/97298332-b65cc600-186c-11eb-83d4-4e3316c033b6.JPG)

3- Verify that you have 5 pods running post the ```oc scale``` command.<br>
```
oc get pods -n guestbook-project
```
![get pods rollout](https://user-images.githubusercontent.com/36239840/97298243-96c59d80-186c-11eb-8286-ef6db2cc02da.JPG)
![pods rollout](https://user-images.githubusercontent.com/36239840/97298286-aa710400-186c-11eb-92e6-3ad2b0cd848f.JPG)
## Update Application
Red Hat OpenShift allows you to do rolling upgrade of your application to a new container image. This allows you to easily update the running image and to easily undo a rollout if a problem is discovered during or after the deployment. In this section, you will be upgrading the image with v1 tag to a new version with v2 tag.<br>
1- Update your deployment using the v2 image.<br>
```
oc set image deployment/myguestbook guestbook=ibmcom/guestbook:v2
```
2- Check the status of the rollout.<br>
```
oc rollout status deployment/myguestbook
```
![rollout v2](https://user-images.githubusercontent.com/36239840/97298915-924db480-186d-11eb-843d-efd2f98f2a7b.JPG)

3- Get a list of pods (newly launched) as part of moving to the v2 image.<br>
```
oc get pods -n guestbook-project
```
![get pods v2](https://user-images.githubusercontent.com/36239840/97298980-aa253880-186d-11eb-9ff1-7a0038c0f2d5.JPG)

4- Copy the name of one of the new pods and use ```oc describe pod``` to confirm that the pod is using the v2 image like the screenshot shown below.<br>
```
oc describe pod <pod-name>
```
![describe pod](https://user-images.githubusercontent.com/36239840/97299403-55ce8880-186e-11eb-8929-4c58a4fdd303.JPG)
Notice that the service and router objects remain unchanged when you changed the underlying docker image version of your Pod.<br>
5- Do a hard refresh on the public router URL to see that the Pod is now using the v2 image of the guestbook application.<br>
![guestbook app](https://user-images.githubusercontent.com/36239840/97299697-cfff0d00-186e-11eb-99e8-28e0cacfafc3.JPG)
## Roll back Applicatoin
When doing a rollout, you can see references to old replicas and new replicas. In this project, the old replicas are the original 5 pods you deployed in the second section when you scaled the application. The new replicas come from the newly created pods with the new image. All these pods are owned by the deployment. The deployment manages these two sets of pods with a resource called ReplicaSet. 
1- To see the guestbook ReplicaSets use the following command.<br>
```
oc  get replicasets -l app=myguestbook
```
![get replicas](https://user-images.githubusercontent.com/36239840/97300667-35073280-1870-11eb-96a4-6bb7d615b87b.JPG)
2- Undo your latest rollout using the following command.<br>

```
oc rollout undo deployment/myguestbook
```
![undo deployment](https://user-images.githubusercontent.com/36239840/97300024-4dc31880-186f-11eb-8ea7-9f68d8840699.JPG)
3- Get the status of undo rollout to see the newly created Pods as part of undoing the rollout.<br>
```
oc rollout status deployment/myguestbook
```
![rollout success](https://user-images.githubusercontent.com/36239840/97300153-7d722080-186f-11eb-874b-e887c0d0b815.JPG)
4- Get the list of the newly created pods as part of undoing the rollout.<br>
```
oc get pods
```
![rollout get pods](https://user-images.githubusercontent.com/36239840/97300243-a0043980-186f-11eb-9743-dc8a4998b1cb.JPG)
5- Copy the name of one of the pods and use it in the ```oc describe pod``` command to view the image and its version used in the pod.<br>
```
oc describe pod <pod-name>
```
![get pods v1](https://user-images.githubusercontent.com/36239840/97300462-efe30080-186f-11eb-9561-209aa7c3471f.JPG)
6- After undoing the rollout, you can check the ReplicaSets and will notice that the old replica set is now active and manages the 5 pods using the following command.<br>
```
oc get replicasets -l app=myguestbook
```
![get replicas 2](https://user-images.githubusercontent.com/36239840/97300861-6b44b200-1870-11eb-81e8-cc8acfd710d6.JPG)
7- To view changes, make sure to hard refresh your browser and pointing and the URL to see if v1 version of the application running.<br>
![guestbook v1 app](https://user-images.githubusercontent.com/36239840/97300913-7e578200-1870-11eb-9e91-81050b709574.JPG)

## Summary
