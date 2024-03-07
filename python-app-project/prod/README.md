
# Deploy My Flask App in k8s

## Instalar APP
```bash
kubectl apply -n prod-environment  -f python-app-project/prod/1_init_db_configmap.yaml
kubectl describe  deployment my-flask-app
kubectl get pods -n uat-environment  -l app=my-flask-app
kubectl logs -n uat-environment  -l app=my-flask-app
```

## INIT  DB
```bash
DB_NAME=knights
 # To drop a db
export POSTGRES_PASSWORD=$(kubectl get secret --namespace uat-environment uat-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

kubectl run uat-postgresql-client --rm --tty -i --restart='Never' --namespace uat-environment --image registry-1.docker.io/bitnami/postgresql:16.2.0-debian-11-r15 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
 --command -- dropdb -f ${DB_NAME} --if-exists --host uat-postgresql -U postgres  -p 5432 

  # To create a db
kubectl run uat-postgresql-client --rm --tty -i --restart='Never' --namespace uat-environment --image registry-1.docker.io/bitnami/postgresql:16.2.0-debian-11-r15 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
 --command -- psql --host uat-postgresql -U postgres  -p 5432  -tc "create database ${DB_NAME}"


 kubectl run uat-postgresql-client --rm --tty -i --restart='Never' --namespace uat-environment --image registry-1.docker.io/bitnami/postgresql:16.2.0-debian-11-r15 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
 --command -- psql --host uat-postgresql -U postgres -d ${DB_NAME} -p 5432 -f ./ddl/init.sql
```


## Instalar APP Service
```bash
kubectl apply -n uat-environment  -f 3_my-app-service.yaml
## TTI curl service
kubectl run mycurlpod -n uat-environment --image=curlimages/curl -rm --tty -- sh
# RUN curl to service
kubectl run mycurlpod -n uat-environment --image=curlimages/curl  --restart=Never -- curl my-flask-app-svc.uat-environment:5000

# INGRESS

# NODEPORT
kubectl apply -n uat-environment  -f ...


```


