Commands:

Starting Minikube :  minikube start
If started then you can check with :  kubectl get nodes
To do this, we can expose the pod to the public internet using the kubectl expose command : `kubectl expose deployment helloworld --type=NodePort`

you can use a `--type=LoadBalancer` that will provision an external IP address would be provisioned to access the service.

To view the final user interface, use the minikube service command. : 'minikube service helloworld' ( minikube <service> <service name>
				

To look at the deployment YAML that runs the all applications, run  kubectl get deployment -o yaml , for single kubectl get deployment/helloworld -o yaml
allowed formats are: custom-columns,custom-columns-file,go-template,go-template-file,json,jsonpath,jsonpath-as-json,jsonpath-file,name,template,templatefile,wide,yaml

The Kubernetes service also comprises YAMLs. Let's take a look at that by running  `kubectl get service helloworld-service -o yaml`.

Notice the `---` that marks the end of one section and starts another.
Example :  contains both deployment and service file.
	apiVersion: apps/v1
	Kind: deployment
	
	-----
	apiVersion: apps/v1
	Kind: service

WORKLOADS TYPES:
Deployment :  A deployment is responsible for keeping a set of pods running.
CronJobs :
Deamon Sets:
Jobs :
Pods :
Replica Sets :
Replication Controllers :
Stateful Sets:

SERVICE TYPES:
Services: A service is responsible for enabling network access to a set of pods.
Ingress:

CONFIG AND STORAGE TYPES:
Config Maps:
Persistent Volume claims
Secrets

To run a replica set for our hello world deployment, run the command `kubectl scale --replicas=3 deploy/helloworld`
This will add three replicas for our deployment, which effectively means three pods running for a single deployment.

LABELS
kubectl get services/deployments/all --show-labels
kubectl get all --show-labels
kubectl label po/helloworld app=helloworldapp --overwrite   ( editing label)
kubectl label po/helloworld app-                            ( delete label using '-')

Searching with label 
kubectl get pods --selector env=production  
kubectl get pods -l 'release-version notin (1.0,2.0)  ( search using notin/in)
kubectl get pods -l 'release-version'                    ( search using label name)
kubectl get pods -l dev-lead!=karthik,env=staging     ( using NOT as '!' )
kubectl delete pods -l dev-lead=karthik               ( deleting pods by label)

Application health checks
A readiness probe is used to know when a container is ready to start accepting traffic.
	containers:
	      - name: helloworld
	        image: karthequian/helloworld:latest
	        ports:
	        - containerPort: 80
	        readinessProbe:
	          # length of time to wait for a pod to initialize
	          # after pod startup, before applying health checking
	          initialDelaySeconds: 10
	          # Amount of time to wait before timing out
	          initialDelaySeconds: 1
	          # Probe for http
	          httpGet:
	            # Path to probe
	            path: /
	            # Port to probe
	            port: 90
	
A liveness probe is used to know when a container might need to be restarted.
	 containers:
	      - name: helloworld
	        image: karthequian/helloworld:latest
	        ports:
	        - containerPort: 80
	        livenessProbe:
	          # length of time to wait for a pod to initialize
	          # after pod startup, before applying health checking
	          initialDelaySeconds: 10
	          # How often (in seconds) to perform the probe.
	          periodSeconds: 5
	          # Amount of time to wait before timing out
	          timeoutSeconds: 1
	          # Kubernetes will try failureThreshold times before giving up and restarting the Pod
	          failureThreshold: 2
	          # Probe for http
	          httpGet:
	            # Path to probe
	            path: /
	            # Port to probe
	            port: 90
	
ROLL OUT
kubectl describe po   ( to check failures in pods )

`kubectl create -f helloworld-black.yaml --record`  (--record records our rollout history )

To update the image for existing deployment, I run this command: 
`kubectl set image deployment/navbar-deployment helloworld=karthequian/helloworld:blue`

We can also take a look at the rollout history by typing `kubectl rollout history deployment/navbar-deployment`.

To rollback the deployment to previous version, we use the rollout undo command `kubectl rollout undo deployment/navbar-deployment`.
Adding  this `--to-revision=version` makes to rollout to the specific version you want to.
 kubectl rollout undo deployment/navbar-deployment --to-revision=2


# Basic troubleshooting techniques
`kubectl describe deployment bad-helloworld-deployment`
`kubectl get pods`
`kubectl describe pod bad-helloworld-deployment-7bb4b7466-f6nkm` (you can check any parameter and EVENTS happening in pods)

`kubectl logs <pod_name>`  (for LOGS on pods)

### Executing commands in a container
`kubectl exec -it <pod-name> -c <container-name> /bin/bash`


### Run the Kubernetes Dashboard in minikube

`minikube addons list`
`minikube addons enable dashboard`
`minikube addons enable metrics-server`
`minikube dashboard`

# Working with config maps
Applications require a way for us to pass data to them that can be changed at deploy time. Examples of this might be log-levels or urls of external systems that the application might need at startup time. Instead of hardcoding these values, we can use a configmap in kubernetes, and pass these values as environment variables to the container.

To create a config map for this literal type `kubectl create configmap logger --from-literal=log_level=debug`
To see all your config maps: `kubectl get configmaps`
To read the value in the logger configmap: `kubectl get configmap/logger -o yaml`
To edit the value, we can run `kubectl edit configmap/logger`

# Working with secrets
Just like configuration data, applications might also require other data that might be of more sensitive in nature- for example database passwords, or API tokens. Passing these in the yaml for a deployment or pod would make them visible to everyone.
In these usecases, use a secret to encapsulate sensitive data.
To create a secret: `kubectl create secret generic apikey --from-literal=api_key=123456789`
Notice that we can't read the value of the secret directly:
`kubectl get secret apikey -o yaml`
Example:
	    spec:
	      containers:
	      - name: secretreader
	        image: karthequian/secretreader:latest
	        env:
	        - name: api_key
	          valueFrom:
	            secretKeyRef:
	              name: apikey
	              key: api_key


### How to run jobs
Jobs are a construct that run a pod once, and then stop. However, unlike pods in deployments, the output of the job is kept around until you decide to remove it.
Running a job is similar to running a deployment, and we can create this by `kubectl create -f simplejob.yaml`
To see the output of the job: `kubectl get jobs`
You can find the pod that ran by doing a `kubectl get pods`, and then get the logs from it as well.

### How to run cron jobs
Cron jobs are like jobs, but they run periodically.
Start your cron by running `kubectl create -f cronjob.yaml`
We can use the cronjob api to view your cronjobs: `kubectl get cronjobs`. It adds the last schedule date
Example: Cronjob.yaml 
	apiVersion: batch/v1beta1
	kind: CronJob
	metadata:
	  name: hellocron
	spec:
	  schedule: "*/1 * * * *" #Runs every minute (cron syntax) or @hourly.
	  jobTemplate:
	    spec:
	      template:
	        spec:
	          containers:
	          - name: hellocron
	            image: busybox
	            args:
	            - /bin/sh
	            - -c
	            - date; echo Hello from your Kubernetes cluster
	          restartPolicy: OnFailure #could also be Always or Never
	  suspend: false #Set to true if you want to suspend in the future
	

## Daemonsets
Daemonsets: A DaemonSet ensures that all Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. Examples of a daemon set would be running your logging or monitoring agent on your nodes.
First, you need make sure you tag minikube with a label of `kubectl label node/minikube infra=development` to label the node first.
In the example, I will just run a simple busybox image as a daemonset, and then run daemonset examples to show how you can tag things to run on specific nodes
`kubectl create -f daemonset.yaml` will run on the nodes
`kubectl create -f daemonset-infra-development.yaml` will only run on nodes labeled `infra=development`(as shown above in the label)
`kubectl create -f daemonset-infra-prod.yaml`  will only run on nodes labeled `infra=production`

## Statefulsets
Statefulsets Manages the deployment and scaling of a set of Pods, and provides guarantees about the ordering and uniqueness of these Pods. Unlike a Deployment, a StatefulSet maintains a sticky identity for each of their Pods.
