# OpenEBS Helm Charts

## Prerequisites
- Install Helm
  You can find instructions to install Helm [here.](https://github.com/helm/helm#install)
  Linux users can run this installation script:
  ```
  curl https://raw.githubusercontent.com/helm/helm/master/scripts/get > get_helm.sh
  chmod 700 get_helm.sh
  ./get_helm.sh
  ```

- Package latest version of openebs 
  ```
  git clone https://github.com/openebs/openebs.git
  helm package openebs/k8s/charts/openebs
  ```
  
  Note the _tgz_ file created. 

## Update with new openebs chart

```
git clone https://github.com/openebs/charts.git
cd charts
mv ../openebs-*.tgz ./docs
helm repo index docs --url https://openebs.github.io/charts
```

## Testing the new chart in minikube

```
vagrant@minikube-dev:~$ rm -rf ~/.helm && helm init --client-only
Creating /home/vagrant/.helm 
Creating /home/vagrant/.helm/repository 
Creating /home/vagrant/.helm/repository/cache 
Creating /home/vagrant/.helm/repository/local 
Creating /home/vagrant/.helm/plugins 
Creating /home/vagrant/.helm/starters 
Creating /home/vagrant/.helm/cache/archive 
Creating /home/vagrant/.helm/repository/repositories.yaml 
$HELM_HOME has been configured at /home/vagrant/.helm.
Not installing Tiller due to 'client-only' flag having been set
Happy Helming!
```

```
vagrant@minikube-dev:~$ helm repo add openebs-charts https://openebs.github.io/charts/
"openebs-charts" has been added to your repositories
vagrant@minikube-dev:~$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Skip local chart repository
...Successfully got an update from the "openebs-charts" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈ 
```

Verify the latest chart version is listed first.

```
vagrant@minikube-dev:~$ cat ~/.helm/repository/cache/openebs-charts-index.yaml
apiVersion: v1
entries:
  openebs:
  - apiVersion: v1
    appVersion: 0.4.0
    created: 2017-10-31T11:36:51.573727018Z
    description: Containerized Storage for Containers
    digest: 61d91376b97883d7c01628f026ee8c4b7c3f25da2dc65affb1a4b6c958cd3f03
    home: http://www.openebs.io/
    icon: https://raw.githubusercontent.com/openebs/chitrakala/master/logo/png/logo-01.png
    keywords:
    - cloud-native-storage
    - block-storage
    - iSCSI
    - storage
    name: openebs
    sources:
    - https://github.com/openebs/openebs
    urls:
    - https://openebs.github.io/charts/openebs-0.4.0.tgz
    version: 0.4.0
generated: 2017-10-31T11:36:51.572803046Z
vagrant@minikube-dev:~$ 
```

For testing the RBAC, tiller needs to be granted access to create openebs namespace and service accounts.

```
kubectl -n kube-system create sa tiller
kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
kubectl -n kube-system patch deploy/tiller-deploy -p '{"spec": {"template": {"spec": {"serviceAccountName": "tiller"}}}}'
kubectl -n kube-system patch deployment tiller-deploy -p '{"spec": {"template": {"spec": {"automountServiceAccountToken": true}}}}'
```

```
vagrant@minikube-dev:~$ helm install openebs-charts/openebs
NAME:   loopy-fox
LAST DEPLOYED: Tue Oct 31 11:43:43 2017
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1beta1/Deployment
NAME                 DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
maya-apiserver       1        1        1           0          1s
openebs-provisioner  1        1        1           0          1s

==> v1/StorageClass
NAME                TYPE
openebs-cassandra   openebs.io/provisioner-iscsi  
openebs-es-data-sc  openebs.io/provisioner-iscsi  
openebs-jupyter     openebs.io/provisioner-iscsi  
openebs-kafka       openebs.io/provisioner-iscsi  
openebs-mongodb     openebs.io/provisioner-iscsi  
openebs-percona     openebs.io/provisioner-iscsi  
openebs-redis       openebs.io/provisioner-iscsi  
openebs-standalone  openebs.io/provisioner-iscsi  
openebs-standard    openebs.io/provisioner-iscsi  
openebs-zk          openebs.io/provisioner-iscsi  

==> v1/ServiceAccount
NAME                   SECRETS  AGE
openebs-maya-operator  1        1s

==> v1beta1/ClusterRole
NAME                   AGE
openebs-maya-operator  1s

==> v1beta1/ClusterRoleBinding
NAME                   AGE
openebs-maya-operator  1s

==> v1/Service
NAME                    CLUSTER-IP  EXTERNAL-IP  PORT(S)   AGE
maya-apiserver-service  10.0.0.233  <none>       5656/TCP  1s


NOTES:
The OpenEBS has been installed. Check its status by running:
  kubectl get pods -n default

To use OpenEBS Volumes, your nodes should have the iSCSI initiator installed. 
Please visit http://docs.openebs.io/ for instructions


vagrant@minikube-dev:~$ 
```


## References
- https://github.com/kubernetes/helm/blob/master/docs/chart_repository.md
