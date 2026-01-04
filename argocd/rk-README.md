YT - DevOps Hobbies - argocd playlist

GIT:
https://github.com/devopshobbies/argocd-tutorial


https://argoproj.github.io/argo-rollouts/features/bluegreen/
https://hub.docker.com/r/argoproj/rollouts-demo/tags
docker pull argoproj/rollouts-demo:blue

_______________________________________________________
YT- 04 - Argocd project & Defining Argocd projects using manifests and terraform

/home/raf/gitRaf/from-git/argocd/v04-customProject

kubectl get appprojects |c

https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/




_______________________________________________________
YT- 06 - Argocd sync policies and sync options
https://argo-cd.readthedocs.io/en/latest/user-guide/sync-options/

  syncPolicy:
  # automated: {}  sync is refreshed ever 6 min this allows synk when refreshed
    automated:      # this allows delete resources as well when syced
      prune: true
	  
[30:30]
you can add anotations to resources like svc etc 

[...]
kind: ServiceAccount
metadata:
	name: something
	annotation:
		argocd.argoproj.io/sync-options: Prune=false
[...]

[41:00]
Selective Sync

apiVersion: argoproj.io/v1alpha1
kind: Application
spec:
  syncPolicy:
    syncOptions:
    - ApplyOutOfSyncOnly=true   [...]
  syncPolicy:
    syncOptions:
    - Replace=true  # Can be used in Application and resource level
	
	
	
# if there are tousands of resources this syncs only selected resources

[50:30]
apiVersion: argoproj.io/v1alpha1
kind: Application
spec:
  syncPolicy:
    syncOptions:
    - FailOnSharedResource=true

2 applications using the same resources in the same ns


_______________________________________________________
07 - sync phases and sync waves		Raf: Skipped it after getting what it is

https://www.youtube.com/watch?v=zIHe3EVp528

argocd.argoproj.io/hook: PreSync
argocd.argoproj.io/sync-wave: "5"			sync wave

argocd hooks

healthcheck example
https://github.com/devopshobbies/argocd-tutorial/blob/main/v07-argocd-sync-phases-waves/sync-phases-waves-manifests-examples/pre-job.yml


apiVersion: batch/v1
kind: Job
metadata:
  name: health-check-pre-job
  annotations:
    argocd.argoproj.io/hook: PreSync
    argocd.argoproj.io/hook-delete-policy: HookFailed
spec:
  template:
    spec:
      containers:
        - name: health-check
          image: curlimages/curl
          command: ["/bin/sh", "-c"]
          args: ["if curl -s http://myapp.local:5000 | grep -qi 'running'; then echo 'Demo Application is up and Running'; exit 0; else echo 'Demo Application is NOT accessible'; exit 1; fi"]
      restartPolicy: Never
  backoffLimit: 0
  
 
 _______________________________________________________
08 - adding cluster & tracking strategies  		RAF: Skipped

Managing multile clusters from single argocd
_______________________________________________________
09 - Argocd applicationSet Part-1				RAF: Skipped
10 - Argocd applicationSet part-2

About managing applications across many clusters

_______________________________________________________
11 - Argo-rollouts overview & Installation

Deployment strategies:
	- Kubernetes native:
		- Rolling-update
		- Recreate
	- Argo-rollouts:
		- Rolling-update
		- Blue/Green
		- Canary

Components of Argo-rollouts:
	- controller
	- resource
	- ingress/service
	- AnalisisTemplate		# link to metric provider which decides if success or rollback the new deployment
	- Metrics Providers



curl -LO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64	
chmod +x ./kubectl-argo-rollouts-linux-amd64
sudo mv ./kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts
kubectl argo rollouts version
	
	
kubectl argo rollouts get rollout rollouts-demo --watch

kubectl argo rollouts promote rollouts-setweight
kubectl argo rollouts abort rollouts-setweight
_______________________________________________________
14 - Analysis In Argo-rollouts

CRDs:
	- AnalysisTemplate				>>
	- ClusterAnalysisTemplate		>> not limited to a namespace
	- AnalysisRun
	
Analysis in B/G stratedgy
	- PrePromotion						Job provider; web provider
	- PostPromotion

=Job provider - a shell script; 

=Web provider - this provider makes a request to url (GET|PUT|POST) and answer has to be json content

{"data": {"ok": true}}

successCondition: .data.ok=true
failureCondition: .data.ok=false
	
	
Basically the job is configured as a healthcheck and if it fails rollback to previous version of the Application

He creates simple healthcheck script and makes a container out of it
[1]--------------------------------------------------------------------------------
	app-success-rate.sh		(chmod u+x)
#!/bin/bash
success=0
failure=0
##################
for i in {1..5}
do
        response=$(curl -s -o /dev/null -w "%{http_code}" $service_url)
        if [ "$response" -eq 200 ]
        then
		((success++))
         else
                ((failure++))
         fi
         sleep 5s
done
##################
echo "Success rate: $success/5" 
##################
if [ "$success" -ge 3 ]
then
        exit 0
else
        exit 1
fi
[2]--------------------------------------------------------------------------------
FROM alpine:latest
RUN apk add --no-cache bash curl
workdir /devops-hobbies
COPY app-success-rate.sh .
[3]--------------------------------------------------------------------------------	
[...]
containers:
	- name: success-rate
	  image: Analysis
	  env:								>> to pass service_url as argument to script
		- name: service_url
		  value: "http://rollout-bluegreen-preview.default.svc.cluster.local:5000"						>> url of previous service
	  command: ["/bin/bash","-c" ]
	  args:
		- ./app-success-rate.sh



_______________________________________________________
15 - Argo-workflows: getting started
https://www.youtube.com/watch?v=UgpVX5Sn5OI&list=PLYrn63eEqAzYttcyB6On1oH35O5rxgDt4&index=14

argo workflows is like mini jenkins running on k8s

6:00  he shows how to customize default variables in helm and yaml files

		ingress is an example here
		how to create values.yaml file
------
18:00 creating roles and rolebinding		>>> granting default sa admin privileges

kubectl create rolebinding default-admin --clusterrole=admin --serviceaccount=argo:default -n argo
						   <binding name>									  <ns  : sa>	

kubectl get rolebindings |c

argo will use sa for executing workflows
------

