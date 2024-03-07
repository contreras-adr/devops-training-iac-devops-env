
# Deploy My Flask App in k8s

## Instalar DB
```bash
echo -n "rootpwd" | base64
kubectl apply -n uat-environment -f python-app-project/uat/1_database-config.yaml 
kubectl apply -n uat-environment -f python-app-project/uat/2_database-root-user-secret.yaml 
kubectl apply -n uat-environment -f python-app-project/uat/3_database-pvc.yaml

kubectl apply -n uat-environment -f python-app-project/uat/4_database-deployment.yaml
kubectl logs -n uat-environment -l app=database
kubectl exec -n uat-environment -l app=database -- cat docker-entrypoint-initdb.d/initdb.sql

kubectl apply -n uat-environment -f python-app-project/uat/5_database-service.yaml
```

## TEST  DB
```bash
DB_NAME=knights

PG_USER=$(kubectl get secret --namespace uat-environment db-root-user-secret  -o jsonpath="{.data.user}" | base64 -d)
echo "PG_USER: $PG_USER"
POSTGRES_PASSWORD=$(kubectl get secret --namespace uat-environment db-root-user-secret  -o jsonpath="{.data.password}" | base64 -d)
echo "POSTGRES_PASSWORD: $POSTGRES_PASSWORD"

kubectl run -it --rm --image=mysql:5.7  -n uat-environment --restart=Never mysql-client -- mysql -h database-svc.uat-environment -u $PG_USER -p$POSTGRES_PASSWORD
```


## Instalar APP Service
```bash
kubectl apply -n uat-environment  -f python-app-project/uat/6_my-app-deployment.yaml 
kubectl apply -n uat-environment  -f python-app-project/uat/7_my-app-service.yaml
## TTI curl service
kubectl run mycurlpod -n uat-environment --image=curlimages/curl -it --rm --restart=Never -- sh
# RUN curl to service
kubectl run mycurlpod -n uat-environment --image=curlimages/curl -it --rm  --restart=Never -- curl my-flask-app-svc.uat-environment:5000

# INGRESS

# NODEPORT
kubectl apply -n uat-environment  -f ...


```


