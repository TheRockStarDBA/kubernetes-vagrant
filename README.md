# Vagrant For K8's

This vagrant script shall install kubernetes cluster on Linux/Windows using vagrant and Virtualbox.

## Dependencies

  - Vagrant.require_version >= 2.0.0
  - Virtualbox
 

# VirtualBox 

Download virtual box for windows from https://www.virtualbox.org/wiki/Downloads 
follow the onscreen steps to install


# Install Vagrant On windows

Download vagrant from https://www.vagrantup.com/downloads.html

after it's installed completely you can test using the command ```vagrant``` in command prompt

```we dont need to install ansible as ansible would be automatically be installed on the master node and used to manage other nodes```

# Vagrant File Parameters

Setup these parameters in vagrant file to select no of node and memory and other specifications  

 - $num_instances = 4 ## Number of Instances Vagrant shall create on your Virtual Box ##

 - $instance_name_prefix = "k8s" ## Your VM Names Prefix ##

 - $vm_memory = 3500 ## VM Memory ##

 - $vm_cpus = 3 ## VM CPU Core to be assigned ##

 - $subnet = "172.17.8" ## VM Subnet ##

 - $os = "ubuntu" ## VM Operating System ##

 - $network_plugin = "flannel" ## Kubernetes Network Plugin

 - $etcd_instances = 3 ## Note: etcd instance should not be even number ##

 - $kube_node_instances = $num_instances #it would be same as no of instances or you could edit with your number #

# K8's installation using vagrant

```
cd kubernetes-vagrant
vagrant up 

vagrant status 
```

### Connect to VM

```
vagrant ssh node_name
```

| VM            |Role      | IP Address    |Box               |
| ------------- |----------| ------------- |------------------|
| k8s-01        | master   | 172.17.8.101  |bento/ubuntu-16.04|
| k8s-02        | node     | 172.17.8.102  |bento/ubuntu-16.04|
| k8s-0X        | node     | 172.17.8.10X  |bento/ubuntu-16.04|
| mssql-jump    | jump-box | 172.17.8.201  |bento/ubuntu-16.04|  

NFS storage provionser is installed with storage class named nfs. along with it microsoft sql server is also installed using helm chart 

Confirm your installation by running a command ( you need to ssh into the master )
```bash
 $ kubectl get nodes
 $ kubectl get pods
 $ kubectl get storageclass
 $ kubectl get pvc
 $ kubectl get pods --all-namespaces
```


For helm list use 
```bash 
$ helm ls
```

### Connect to Dashboard

```
https://172.17.8.101:30910/#!/login
```

Generate Key
```
kubectl -n kube-system describe secrets \
`kubectl -n kube-system get secrets | awk '/clusterrole-aggregation-controller/ {print $1}'` \
| awk '/token:/ {print $2}'
```
