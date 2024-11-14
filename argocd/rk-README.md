YT - DevOps Hobbies - argocd playlist

GIT:
https://github.com/devopshobbies/argocd-tutorial

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
15 - Argo-workflows: getting started
https://www.youtube.com/watch?v=UgpVX5Sn5OI&list=PLYrn63eEqAzYttcyB6On1oH35O5rxgDt4&index=14


6:00  he shows how to customize default variables in helm and yaml files

		ingress is an example here
		how to create values.yaml file

18:00 creating roles and rolebinding