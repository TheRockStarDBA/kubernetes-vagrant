# Vagrant For K8's

This vagrant script shall install kubernetes cluster on your virtualbox .
## Dependencies
  - Vagrant.require_version >= 2.0.0
  - Headless virtualbox
  - Ansible

# VirtualBox Headless Installation

Step 1: Update Ubuntu

Before installing VirtualBox, run the commands below to update the Ubuntu server.   

```bash
$ sudo apt-get update && sudo apt-get dist-upgrade && sudo apt-get autoremove
```

Step 2: Install Required Linux Headers

Now that your system is updated, run the commands below to install required Ubuntu linux headers.

 ```bash
$ sudo apt-get -y install gcc make linux-headers-$(uname -r) dkms
 ```

Step 3: Add VirtualBox Repository and key

After installing the required package above, run the commands below to install VirtualBox repository key.
```bash
 $ wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
 $ wget -q https://www.virtualbox.org/download/oracle_vbox.asc -O- | sudo apt-key add -
 ```
 Next run the commands below to add VirtualBox repository to your system.

 ```bash
 $ sudo sh -c 'echo "deb   http://download.virtualbox.org/virtualbox/debian $(lsb_release -sc) contrib" >> /etc/apt/sources.list'
```
Step 4: Install VirtualBox

After adding the repository and key, run the commands below to install VirtualBox 5.2. At the time of this writing the latest version of the software was 5.2. If there are newer versions available, please replace the 2 below with the current latest.
```bash
 $ sudo apt-get update
 $ sudo apt-get install virtualbox-5.2
```
To verify if VirtualBox is installed, run the commands below.

 ``` VBoxManage -v```

# Install Ansible
```bash
 $ sudo apt-get install software-properties-common
 $ sudo apt-add-repository ppa:ansible/ansible
 $ sudo apt-get update
 $ sudo apt-get install ansible
```
# Install Vagrant On Ubuntu 16.04
```bash
 $ sudo apt-get update
 $ apt-cache show vagrant
 $ sudo apt-get install vagrant
```
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

 - go inside the  directory where vagrant file is present.
 - run the command ```vagrant up```
 - after the installation completes run the command ``` vagrant status ```
 - In order to ssh into the VM's created run the command ``` vagrant ssh node_name```

Confirm your installation by running a command ( you need to ssh into the master )
```bash
 $ kubectl get nodes
 $ kubectl get pods --all-namespaces
```
