# Heroku-like deploy on kubernetes

This is supposed to be a POC of joining the best of all worlds: the Heroku-like deploy we all love with the price of running on a VM with kubernetes orchestration.

Tools that make this possible:
- [Kubernetes](https://github.com/kubernetes/kubernetes)
- [Deis Workflow](https://github.com/deis/workflow)

## Steps to reproduce the environment:

**Create kubernetes cluster on [GCP](https://cloud.google.com/)**
``` bash
gcloud container clusters create CLUSTER_NAME --zone CLUSTER_ZONE --num-nodes=3 --machine-type=f1-micro
```

**Install [Deis](https://github.com/deis/workflow)**
``` bash
curl -sSL http://deis.io/deis-cli/install-v2.sh | bash # Download Deis
sudo mv $PWD/deis /usr/local/bin/deis # Move to place in $PATH
deis version # Verify installation in successful
```

**Install [Helm](https://github.com/kubernetes/helm)**
``` bash
wget https://kubernetes-helm.storage.googleapis.com/helm-v2.5.0-linux-386.tar.gz -O helm.tar.gz
# tar -xzvf helm.tar.gz + mv executable to $PATH location
helm version
```
**Install [Workflow](https://deis.com/docs/workflow/) on cluster**
``` bash
kubectl create sa tiller-deploy -n kube-system
kubectl create clusterrolebinding helm --clusterrole=cluster-admin --serviceaccount=kube-system:tiller-deploy
helm init --service-account=tiller-deploy
helm repo add deis https://charts.deis.com/workflow
sleep 10 # Wait a bit until there is a Tiller Pod available
helm install deis/workflow --namespace deis
```
Might take 10 minutes or so until all pods are running. Check with ```watch kubectl get pods --namespace=deis```

**Register Deis admin user**

Find LoadBalancer IP:
``` bash
kubectl --namespace=deis describe svc deis-router | grep LoadBalancer
```
Define hostname env var, which is composed by the LoadBalancer IP + ".nip.io", e.g. "35.184.245.126.nip.io"

Effectively register the user:
``` bash
deis register http://$hostname
```
**Create the application on Deis**
``` bash
deis create
```
The application name will be informed in the output and a remote will be added to deploy the application, like so:
``` bash
git push deis master
```
