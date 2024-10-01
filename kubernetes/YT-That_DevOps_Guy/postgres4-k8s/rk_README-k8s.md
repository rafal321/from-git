https://github.com/marcel-dempers/docker-development-youtube-series/tree/master/storage/databases/postgresql/4-k8s-basic

kubectl -n postgresql create secret generic postgresql \
  --from-literal POSTGRES_USER="postgresadmin" \
  --from-literal POSTGRES_PASSWORD='admin123' \
  --from-literal POSTGRES_DB="postgres" \
  --from-literal REPLICATION_USER="replicationuser" \
  --from-literal REPLICATION_PASSWORD='replicationPassword'

kubectl exec -it pod/postgres-0 -- bash
ls /data/archive/

psql -U postgresadmin -d postgres
psql -U postgresadmin postgres