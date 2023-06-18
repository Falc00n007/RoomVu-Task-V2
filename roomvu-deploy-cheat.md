







# To deploy the roomvu manifests to a Kubernetes cluster [in my home lab i'd on minikube].

# This is  final step to deploy the manifests to a Kubernetes cluster.
# To do this, you’ll need to use kubectl command.
# Here are the commands you’ll need to run:


# this command run for create & Configure the MySQL database secret 

$ kubectl create secret generic mysql-secret --from-literal=username=cm9vbXZ1Og== --from-literal=password=cm9vbXZ1Og==


# this command run for create a Kubernetes persistent volume to store data of the MySQL data.

$ kubectl apply -f pv-mysql.yaml


# This command used to request storage resources from a Persistent Volume (PV).
# The PVCs provide an abstraction layer between the application and the storage.

$ kubectl apply -f pvc-mysql.yaml

# this is ROOMVU deployment manifest. The deployment manifest defines how many replicas of 
# Roomvu Laravel 10 application should be running.

$ kubectl apply -f roomvu-app-k8s.yaml


# This is The service manifest defines for how to expose roomvu Laravel 10 application to
# other pods in the cluster. it's mean Services are used with load balancing, Port Mapping,
# External Access.

$ kubectl apply -f roomvu-app-service.yaml


# The ingress manifest defines how to expose roomvu Laravel 10 application to external 
# clients. Here’s an example ingress manifest:
# This manifest defines an ingress named roomvu-app that rewrites the target path and
# exposes the service on the host roomvu-app.example.com in my case.
# If you want, change it to you domain name you needed.

$ kubectl apply -f roomvu-app-ingress.yaml

# this file create for Scabality and availability, it's mean scaleup and down environment 
# with having full-scale production cluster. finaly it's for Horizontal Pod Autoscaler, which
#  is a component of Kubernetes that automatically scales the number of pods in a deployment
# or replica set based on observed cpu.
# !!! Atention !!! : In my case, i ran this project on my laptop with lower resources. this 
# project sample is created and configured for smal base hardware. if you need to change it 
# to Pro call me farshidrahimi.ca@gmail.com ro +98 939 6310 462

$ kubectl apply -f roomvu-app-hpa.yaml 