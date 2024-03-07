
# IAC-DEOPS Environment


## CI / CD Strategy with Jenkins and Docker



### Configure Jekins modules to install in *jenkins/plugins.txt*

### Install Jenkins in local environment.
https://github.com/jenkinsci/docker/
```bash
docker-compose up -d jenkins --build --force-recreate

```

### Create GitHub SSH Key for Jenkins
```bash
ssh-keygen -C "contreras.adr@outlook.com" -f ~/.ssh/jenkins-github
cat ~/.ssh/jenkins-github.
```



## Instalar cluster k3d
```bash
k3d cluster create devops-training-cluster \
    --api-port 6443 --servers 1 --agents 1 \
    -p "32760-32767:32760-32767@server:0 -v /tmp/k3d-vol:/tmp/

kubectl cluster-info
kubectl get nodes
```

## Incluir extension vscode-k3d en Visual Studio
https://github.com/k3d-io/vscode-k3d


## LOCAL Persistent Volume Storage
https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/
```bash
# Create Persistent Volume on the clusters node file system
$ kubectl apply -f k3s/local-persistent-volume.yaml 
$ kubectl get pv

## Create environments
```bash
kubectl create ns uat-environment
kubectl create ns prod-environment
```

## STOP/START CLUSTER
```bash
$ k3d cluster stop devops-training-cluster
$ k3d cluster start devops-training-cluster
```

## REMOVE CLUSTER
```bash
$ k3d cluster delete devops-training-cluster
```

## Install ArgoCD
```bash
## Install ArgoCD CI
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64

## Install ArgoCD IN K8S
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl patch deploy argocd-server -n argocd --type json -p '[{"op": "replace", "path": "/spec/template/spec/containers/0/command", "value": ["argocd-server", "--insecure", "--staticassets","/shared/app"]}]'
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

## Get Inital Password
argocd admin initial-password -n argocd
# JGyojFfkS5hVDEyo
# This password must be only used for first time login. 
# We strongly recommend you update the password using `argocd account update-password`.


## Alternative CORE ONLY
# kubectl delete ns argocd --force --grace-period=0
# kubectl create namespace argocd
# kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/core-install.yaml
# argocd login --core

```

## Install postgresql In Environments
```bash
#https://docs.bitnami.com/general/infrastructure/postgresql/
helm install prod-postgresql -n prod-environment oci://registry-1.docker.io/bitnamicharts/postgresql

# PostgreSQL can be accessed via port 5432 on the following DNS names from within your cluster:
#     prod-postgresql.prod-environment.svc.cluster.local - Read/Write connection

# To get the password for "postgres" run:
export POSTGRES_PASSWORD=$(kubectl get secret --namespace prod-environment prod-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)
# YjNRM0taVWw1aw==

# To connect to your database run the following command
kubectl run prod-postgresql-client --rm --tty -i --restart='Never' --namespace prod-environment --image registry-1.docker.io/bitnami/postgresql:16.2.0-debian-11-r15 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
 --command -- psql --host prod-postgresql -U postgres -d postgres -p 5432


DB_NAME=knights
 # To drop a db
kubectl run prod-postgresql-client --rm --tty -i --restart='Never' --namespace prod-environment --image registry-1.docker.io/bitnami/postgresql:16.2.0-debian-11-r15 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
 --command -- dropdb -f ${DB_NAME} --if-exists --host prod-postgresql -U postgres  -p 5432 

  # To create a db
kubectl run prod-postgresql-client --rm --tty -i --restart='Never' --namespace prod-environment --image registry-1.docker.io/bitnami/postgresql:16.2.0-debian-11-r15 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
 --command -- psql --host prod-postgresql -U postgres  -p 5432  -tc "create database ${DB_NAME}"

 ##Execute DDL
 kubectl run prod-postgresql-client --rm --tty -i --restart='Never' --namespace prod-environment --image registry-1.docker.io/bitnami/postgresql:16.2.0-debian-11-r15 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
 --command -- psql --host prod-postgresql -U postgres -d ${DB_NAME} -p 5432 -f ./data/init.sql

 export POSTGRES_PASSWORD=$(kubectl get secret --namespace prod-environment prod-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

```
