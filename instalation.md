go inside folder :-
/media/ashutosh/unix-extra/development/spark-kubernetes



# Deploying Spark on Kubernetes

## Want to learn how to build this?

Check out the [post](https://testdriven.io/deploying-spark-on-kubernetes).

## Want to use this project?

### Minikube Setup

Install and run [Minikube](https://kubernetes.io/docs/setup/minikube/):

1. Install a  [Hypervisor](https://kubernetes.io/docs/tasks/tools/install-minikube/#install-a-hypervisor) (like [VirtualBox](https://www.virtualbox.org/wiki/Downloads) or [HyperKit](https://github.com/moby/hyperkit)) to manage virtual machines
1. Install and Set Up [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) to deploy and manage apps on Kubernetes
1. Install [Minikube](https://github.com/kubernetes/minikube/releases)

Start the cluster:
minikube stop

rm -rf  ~/.minikube
```sh
 minikube start  --extra-config=kubelet.authorization-mode=AlwaysAllow --memory 8192 --cpus 4 --kubernetes-version v1.7.0

minikube dashboard
```
kubectl create secret docker-registry dockersecret --docker-server=https://index.docker.io/v1/ --docker-username=atppath --docker-password=ashu2012 
 


# Install Helm

curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get-helm-3 >  get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh

# Initialize helm, install Tiller(the helm server side component)
helm init
# Make sure we get the latest list of chart
helm repo update
# * Happy Helming * 

helm ls


helm repo list
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo list
helm install redis bitnami/redis

helm delete redis
helm repo remove bitnami

# helm install sparkctl
$ helm repo add spark-operator https://googlecloudplatform.github.io/spark-on-k8s-operator

$ helm install my-release spark-operator/spark-operator --namespace default --set webhook.enable=true


helm status --namespace default my-release
helm delete my-release


# Set up non root docker instatlation and installl password less docker pull after succesfull login
https://github.com/docker/docker-credential-helpers/issues/102

sudo apt-get install pass


wget https://github.com/docker/docker-credential-helpers/releases/download/v0.6.0/docker-credential-pass-v0.6.0-amd64.tar.gz && tar -xf docker-credential-pass-v0.6.0-amd64.tar.gz && chmod +x docker-credential-pass && sudo mv docker-credential-pass /usr/local/bin/

gpg2 --gen-key

pass init "<Your Name>"

sed -i '0,/{/s/{/{\n\t"credsStore": "pass",/' ~/.docker/config.json

docker login


Build the Docker image:

```sh
$ eval $(minikube docker-env)
$ docker build -t spark-hadoop:3.0.0 -f ./docker/Dockerfile ./docker
```

# build docker images as per official spark documentation and use kubernetes
https://www.youtube.com/watch?v=ZzFdYm_DqEM
spark-2.4.7-bin-hadoop2.7$ ./bin/docker-image-tool.sh -r atppath -t spark-2.4.7 buil

## download spark 3.0.0 from https://spark.apache.org/downloads.html
## and build local kubernetes image 
./bin/docker-image-tool.sh -r atppath/spark-3.0.1  -t spark-3.0.1 build


kubectl delete sparkapplications spark-pi 
sparkapplication.sparkoperator.k8s.io "spark-pi" delete

kubectl describe sparkapplications spark-pi 

kubectl apply -f spark-example-deployment.yaml 


# push docker build images to locl repository

ashutosh@atp:/media/ashutosh/unix-extra/development/spark-kubernetes$ docker tag spark-hadoop:3.0.0 atppath/spark-hadoop:3.0.0
ashutosh@atp:/media/ashutosh/unix-extra/development/spark-kubernetes$ docker push atppath/spark-hadoop:3.0.0  
docker push atppath/spark:spark-2.4.7 

# to run spark shell Create the deployments and services:

```sh
sh create.sh
sh delete.sh
```

Add an entry to /etc/hosts:

```sh
$ echo "$(minikube ip) spark-kubernetes" | sudo tee -a /etc/hosts
```

Test it out in the browser at [http://spark-kubernetes/](http://spark-kubernetes/).


minikube addons enable ingress

http://blog.madhukaraphatak.com/scaling-spark-with-kubernetes-part-6/

### not needed if using spark-shell and creat.sh and delte.sh way
### for official spark submit it is needed

kubectl create serviceaccount spark
kubectl create clusterrolebinding spark-role --clusterrole=edit --serviceaccount=default:spark --namespace=default

# service account in spark submit way should have required access
kubernetes$ kubectl auth can-i create pods --as=system:serviceaccount:default:spark 
no
 should be yes

kubectl create clusterrolebinding add-on-cluster-admin --clusterrole=cluster-admin --serviceaccount=spark:default

# env variable for spark submit job command line
export DRIVER_NAME=new-driver
export DOCKER_IMAGE=atppath/spark:spark-3.0.1
export NAMESPACE=default
export SA=spark

# know kubernetes cluster ip
$ kubectl cluster-info
Kubernetes master is running at https://192.168.99.104:8443
KubeDNS is running at https://192.168.99.104:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy



kubectl port-forward spark-master 8080:8080
check localhost:8080 to see progress

kubectl port-forward spark-master-689488fb8-l7cbg 4040:4040


kubectl proxy

spark-master-689488fb8-l7cbg   1/1     Running   0          2d16h
spark-worker-77796cf5b-2t2v8   1/1     Running   0          2d16h
spark-worker-77796cf5b-gb7gf   1/1     Running   0          2d16h
ashutosh@atp:/media/ashutosh/unix-extra/development/spark-2.4.7-bin-hadoop2.




# create a jump pod and run spark submit from there
https://medium.com/@tunguyen9889/how-to-run-spark-job-on-eks-cluster-54f73f90d0bc


spark-submit --name sparkpi-1  \
--master k8s://http://spark-master:7077  \
--deploy-mode cluster  \
--conf spark.kubernetes.driver.pod.name=$DRIVER_NAME  \
--conf spark.kubernetes.container.image=$DOCKER_IMAGE \
--conf spark.kubernetes.container.image.pullPolicy=Never \
--conf spark.driver.host=172.17.0.6  \
--conf spark.kubernetes.kerberos.enabled=false \
--class org.apache.spark.examples.SparkPi \
--conf spark.executor.instances=2 \
local:///opt/spark-3.0.0-bin-hadoop2.7/examples/jars/spark-examples_2.12-3.0.0.jar 10000000


## working below 

### outside from cluster  running as per official spark website

./spark-submit --name sparkpi-1  \
--master k8s://https://192.168.99.104:8443  \
--deploy-mode cluster  \
--conf spark.kubernetes.driver.pod.name=$DRIVER_NAME  \
--conf spark.kubernetes.container.image=$DOCKER_IMAGE \
--conf spark.kubernetes.authenticate.driver.serviceAccountName=$SA \
--conf spark.kubernetes.container.image.pullPolicy=Never \
--conf spark.driver.extraJavaOptions=-Xms100m \
--conf spark.executor.memory=512m \
--conf spark.driver.memory=512m \
--class org.apache.spark.examples.SparkPi \
--conf spark.executor.instances=2 \
local:///opt/spark/examples/jars/spark-examples_2.12-3.0.1.jar 100


### inside master pod using create.sh and delete.sh

./spark-submit --name sparkpi-1  \
--master k8s://http://192.168.99.104:8443  \
--deploy-mode cluster  \
--conf spark.kubernetes.container.image=$DOCKER_IMAGE \
--conf spark.kubernetes.container.image.pullPolicy=Never \
--conf spark.driver.host=172.17.0.10 \
--conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
--conf spark.kubernetes.driver.pod.name=$DRIVER_NAME \
--class org.apache.spark.examples.SparkPi \
--conf spark.executor.instances=2 \
local:///opt/spark-3.0.0-bin-hadoop2.7/examples/jars/spark-examples_2.12-3.0.0.jar 10000000


### Using helm/kubernetes sparkoperator to submit spark job

kubectl describe sparkapplications spark-pi 

kubectl apply -f spark-example-deployment.yaml 


 ## spark shell in pod

https://stackoverflow.com/questions/60210110/how-to-get-access-to-spark-shell-from-kubernetes
 kubectl get services

 kubectl get services
 NAME           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)             AGE
 kubernetes     ClusterIP   10.96.0.1      <none>        443/TCP             3h14m
 spark-master   ClusterIP   10.108.65.18   <none>        8080/TCP,7077/TCP   179m


spark://CoarseGrainedScheduler@spark-master-765dd8df75-mp8dl:36559
# go inside master pod
 	kubectl exec -it <spark pod name> -- /bin/bash

# cjheck master pod ip
cat /etc/hosts

#type below commands to run shell from inside master pod
spark-shell --master spark://spark-master-765dd8df75-mp8dl:7077   --conf spark.driver.bindAddress=<MASTER-POD-IP> --conf spark.driver.host=<MASTER-POD-IP> --total-executor-cores 2 --executor-memory 1024m

# working inside master pod
spark-shell --master spark://spark-master-765dd8df75-965jl:7077   --conf spark.driver.bindAddress=172.17.0.10 --conf spark.driver.host=172.17.0.10 --total-executor-cores 2 --executor-memory 1024m


spark-shell --master spark://spark-master:7077   --conf spark.driver.bindAddress=172.17.0.6 --conf spark.driver.host=172.17.0.6 --total-executor-cores 2 --executor-memory 1024m

# basic command to test shell 
 sc .parallelize(1 to 100,5) . collect


# run bash shell inside  pod for spark submit
 kubectl run my-shell --rm -i --tty --image atppath/spark-hadoop:3.0.0 -- bash
 kubectl run my-shell --rm -i --tty --image atppath/spark:spark-3.0.1 -- bash



# kubernetes sparkoperator to submit spark job

kubectl describe sparkapplications spark-pi 

kubectl apply -f spark-example-deployment.yaml 


# kubernetes sparkoperator to delete submited spark job

kubectl delete sparkapplications spark-pi 
sparkapplication.sparkoperator.k8s.io "spark-pi" delete



#  spark job status

kubectl port-forward spark-master 8080:8080
check localhost:8080 to see progress
kubectl logs -f new-driver
kubectl port-forward spark-master-689488fb8-l7cbg 4040:4040