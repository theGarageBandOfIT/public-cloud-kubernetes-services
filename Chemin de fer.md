# AWS with KOPS

## AWS CLI configuration
```bash
export AWS_PROFILE=skynetvsapes
export KOPS_STATE_STORE=s3://skynetvsapes
```

## S3 bucket configuration
```bash
aws s3api create-bucket --bucket=skynetvsapes --region us-east-1
```

## Cluster creation

```bash
kops create cluster --cloud=aws \
  --dns-zone=bien-chez-vous.info \
  --master-size=t2.medium \
  --master-count=3 \
  --master-zones=us-east-1a,us-east-1b,us-east-1c \
  --network-cidr=10.0.0.0/22 \
  --node-count=3 \
  --node-size=m4.large \
  --zones=us-east-1a,us-east-1b,us-east-1c \
  --name=skynetvsapes-k8s.bien-chez-vous.info
```

```bash
kops update cluster skynetvsapes-k8s.bien-chez-vous.info --yes
```

## Cluster validation
```bash
kops validate cluster skynetvsapes-k8s.bien-chez-vous.info
```

## kubectl configuration injection by kops
```bash
kops export kubecfg
```

# Dashboard installation
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml  --validate=false
```

## Access to dashboard
```bash
kubectl proxy
```

http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/

## Get the right token to access dashboard
```bash
kubectl -n kube-system get secret
kubectl -n kube-system describe secret replicaset-controller-token-kzpmc
```


## Cluster upgrade to 1.8.3
```bash
kops edit cluster skynetvsapes-k8s.bien-chez-vous.info
kops update cluster skynetvsapes-k8s.bien-chez-vous.info --yes
kops rolling-update cluster --yes
```

### kubectl configuration switching

#### Get current context
```bash
kubectl config get-contexts
```

#### Set the right context
```bash
kubectl config set-context
```

# Azure AKS

## Azure CLI configuration
```bash
az login
```

## Resource group creation
```bash
az account list-locations
az group create --name K8Smeetup_RG2 --location "centralus"
```


```bash
az aks create \
    --name K8Smeetup-akscluster2 \
    --resource-group K8Smeetup_RG2 \
    --kubernetes-version 1.7.7 \
    --node-count 2 \
    --node-vm-size Standard_D2_v2 \
    --ssh-key-value /Users/ludovic/OneDrive/Documents/Tech/Azure/credentials/lpiot-tribble_azure_ssh-key.pub

az aks scale --name K8Smeetup-akscluster --resource-group K8Smeetup_RG2 --node-count 3
```


## kubectl configuration injection by Azure CLI
```bash
az aks get-credentials --resource-group K8Smeetup_RG2 --name K8Smeetup-akscluster2
```

## dashboard access
```bash
az aks browse --resource-group K8Smeetup_RG2 --name K8Smeetup-akscluster2
```



# Helm installation

## Helm Client (via HomeBrew)
```bash
brew install kubernetes-helm
```

## Helm Tiller
```bash
helm init
```

## Helm update of the repositories
```bash
helm repo update
```

# App installation

## MySQL installation
```bash
helm install \
    --name apes-mysql \
    --set mysqlRootPassword=toto42,mysqlUser=my-user,mysqlPassword=my-password,mysqlDatabase=symfony_demo \
    stable/mysql
```

## retrieve mySQL root password

```bash
kubectl get secret --namespace default apes-mysql-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo
```

## data injection
```bash
POD=$(kubectl get pods | grep mysql | awk '{print $1}')
```

```bash
kubectl port-forward $POD 3306:3306
```

```bash
cat ../app/data.sql | mysql -p -h127.0.0.1 -uroot
```

## App installation

```bash
helm install --name app ./
```



# Google Container Engine (GKE)

## Cluster creation
```bash
gcloud container clusters create k8smeetup-gke \
    --zone us-central1-a \
    --additional-zones us-central1-b,us-central1-c \
    --cluster-version=1.7.8-gke.0 \
    --enable-autoscaling \
    --min-nodes=2 \
    --max-nodes=3 \
    --num-nodes=3 \
    --enable-autoupgrade
```
## Cluster resizing
```bash
gcloud container clusters resize k8smeetup-gke --size=1 --zone=us-central1-a
```