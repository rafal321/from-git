################################################
# [0] create namespace
kubectl create namespace mysql-cluster
kubectl config set-context --current --namespace mysql-cluster
kubectl config view --minify -o jsonpath='{..namespace}' && echo " "

# [1] custom resource definitions
kubectl apply -f deploy-crds.yaml
kubectl get crd|c
# [2] deploy operator						(namespaced)
kubectl apply -f deploy-operator.yaml

kubectl -n mysql-operator get deployment mysql-operator
kubectl -n mysql-operator get po
kubectl -n mysql-operator describe po <po_name>

# [3] Create secret for root user			(namespaced)
kubectl create secret generic mypwds \
      --from-literal=rootUser=root \
      --from-literal=rootHost=% \
      --from-literal=rootPassword="sakila[]1" --dry-run=client -oyaml
kubectl get secrets |c
kubectl get secret mypwds -oyaml |y

# [4] deploy cluster                        (namespaced)
kubectl apply -f mycluster.yaml


More settings s3 etc
# https://stackoverflow.com/questions/76780191/how-to-initialize-innodb-cluster-created-via-kubernetes-mysql-operator-through-s

ERROR:
Handler 'on_pod_create' failed temporarily: Sidecar of mycluster-0 is not yet configured



[██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████]
MySQL 8.2 is what we were all waiting for for Kubernetes with Transparent Read/Write Splitting
https://sredevops.org/en/mysql-8-2-is-what-we-were-all-waiting-for-for-kubernetes-with-transparent-read-write-splitting/
https://github.com/mysql/mysql-operator/tree/8.4.1-2.1.4/deploy?ref=sredevops.org
https://dev.mysql.com/downloads/mysql/     8.4.2 LTS
Raf: deploy direcly from github
[████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████]

Troubleshooting Common Issues
https://docs.vmware.com/en/VMware-SQL-with-MySQL-for-Kubernetes/1.10/vmware-mysql-k8s/troubleshooting.html

select MEMBER_HOST, MEMBER_STATE, MEMBER_ROLE from performance_schema.replication_group_members;
+------------------------------------------------------------------------+--------------+-------------+
| MEMBER_HOST                                                            | MEMBER_STATE | MEMBER_ROLE |
+------------------------------------------------------------------------+--------------+-------------+
| mycluster-v1-1.mycluster-v1-instances.mysql-cluster3.svc.cluster.local | ONLINE       | SECONDARY   |
| mycluster-v1-0.mycluster-v1-instances.mysql-cluster3.svc.cluster.local | ONLINE       | PRIMARY     |
| mycluster-v1-2.mycluster-v1-instances.mysql-cluster3.svc.cluster.local | ONLINE       | SECONDARY   |
+------------------------------------------------------------------------+--------------+-------------+

select @@gtid_executed;
SELECT * FROM information_schema.processlist WHERE INFO = 'Group replication applier module' ;
[████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████]