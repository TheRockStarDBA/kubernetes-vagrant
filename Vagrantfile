# -*- mode: ruby -*-
# # vi: set ft=ruby :
# Need nfs server package - Example - apt-get install nfs-kernel-server



Vagrant.require_version ">= 2.0.0"



# Uniq disk UUID for libvirt
DISK_UUID = Time.now.utc.to_i

SUPPORTED_OS = {

  "ubuntu"        => {box: "bento/ubuntu-16.04", bootstrap_os: "ubuntu", user: "vagrant"},

}

SUPPORTED_JUMP_OS = {

  "ubuntu"        => {box: "bento/ubuntu-16.04", bootstrap_os: "ubuntu", user: "vagrant"},

}

# Defaults for config options defined in CONFIG
$num_instances = 2
$instance_name_prefix = "k8s"
$vm_gui = false
$vm_memory = 3000
$vm_cpus = 1
$shared_folders = {}
$forwarded_ports = {}
$subnet = "172.17.8"
$os = "ubuntu"
$network_plugin = "flannel"
# The first three nodes are etcd servers
$etcd_instances = 1
# The first two nodes are kube masters
$kube_master_instances = 1
# All nodes are kube nodes
$kube_node_instances = 2
# The following only works when using the libvirt provider
$kube_node_instances_with_disks = false
$kube_node_instances_with_disks_size = "20G"
$kube_node_instances_with_disks_number = 2

$jumpbox_node = 1
$local_release_dir = "/vagrant/temp"

host_vars = {}


$box = SUPPORTED_OS[$os][:box]


Vagrant.configure("2") do |config|
  # always use Vagrants insecure key
  config.ssh.insert_key = true
  config.vm.box = $box
  if SUPPORTED_OS[$os].has_key? :box_url
    config.vm.box_url = SUPPORTED_OS[$os][:box_url]
  end
  config.ssh.username = SUPPORTED_OS[$os][:user]
  # plugin conflict
  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  $num_loop = $num_instances + $jumpbox_node
  (1..$num_loop).each do |i|

    if i == $num_loop && $jumpbox_node != 0 then
	  name = "client-admin"
	  ip = "#{$subnet}.#{i+201}"
	  bootstrap_os = SUPPORTED_OS[$os][:bootstrap_os]
	else
	  name = "%s-%02d" % [$instance_name_prefix, i]
	  ip = "#{$subnet}.#{i+100}"
	  bootstrap_os = SUPPORTED_OS[$os][:bootstrap_os]
	end

	config.vm.define vm_name = name % [$instance_name_prefix, i] do |config|
	  config.vm.hostname = name

	  if $expose_docker_tcp
		config.vm.network "forwarded_port", guest: 2375, host: ($expose_docker_tcp + i - 1), auto_correct: true
	  end

	  $forwarded_ports.each do |guest, host|
		config.vm.network "forwarded_port", guest: guest, host: host, auto_correct: true
	  end


	  config.vm.synced_folder ".", "/vagrant", type: "rsync", rsync__args: ['--verbose', '--archive', '--delete', '-z']
    config.vm.synced_folder "./volumes", "/vagrant/volumes", type: "nfs"
	  $shared_folders.each do |src, dst|
		config.vm.synced_folder src, dst, type: "rsync", rsync__args: ['--verbose', '--archive', '--delete', '-z']
	  end

	  config.vm.provider :virtualbox do |vb|
		vb.gui = $vm_gui
		vb.memory = $vm_memory
		vb.cpus = $vm_cpus
	  end

	  config.vm.provider :libvirt do |lv|
	    lv.memory = $vm_memory
	  end

	  host_vars[vm_name] = {
		"ip": ip,
		"ansible_host": ip,
		"bootstrap_os": bootstrap_os,
		"ansible_port": 22,
		"ansible_user": 'vagrant',
		"ansible_connection": 'ssh',
		"ansible_ssh_user": 'vagrant',
		"ansible_ssh_pass": 'vagrant',
		"local_release_dir" => $local_release_dir,
		"download_run_once": "False",
		"kube_network_plugin": $network_plugin
	  }

	  config.vm.network :private_network, ip: ip
	  config.ssh.username = "vagrant"
	  config.ssh.password = "vagrant"

	  config.vm.network "forwarded_port", guest: 30910 , host: 30910 , auto_correct: true
	  # Disable swap for each vm
	  config.vm.provision "shell", inline: "swapoff -a"
	  config.vm.provision "shell", inline: "apt-get install sshpass python-netaddr -y"

	  config.vm.synced_folder "./tokens", "/vagrant/tokens", type: "nfs"
	  if $kube_node_instances_with_disks
		# Libvirt
		driverletters = ('a'..'z').to_a
		config.vm.provider :libvirt do |lv|
		  # always make /dev/sd{a/b/c} so that CI can ensure that
		  # virtualbox and libvirt will have the same devices to use for OSDs
		  (1..$kube_node_instances_with_disks_number).each do |d|
			lv.storage :file, :device => "hd#{driverletters[d]}", :path => "disk-#{i}-#{d}-#{DISK_UUID}.disk", :size => $kube_node_instances_with_disks_size, :bus => "ide"
		  end
		end
	  end

	  # Only execute once the Ansible provisioner,
	  # when all the machines are up and ready.
  if i == $num_loop
	config.vm.provision "ansible_local" do |ansible|
	  ansible.playbook = "cluster.yml"
	  ansible.install_mode = "pip"
	  ansible.become = true
	  ansible.limit = "all"
	  ansible.raw_arguments = ["--forks=#{$num_loop}", "--flush-cache"]
	  ansible.host_vars = host_vars
	  #ansible.tags = ['download']
	  ansible.groups = {
		"etcd" => ["#{$instance_name_prefix}-0[1:#{$etcd_instances}]"],
		"kube-master" => ["#{$instance_name_prefix}-0[1:#{$kube_master_instances}]"],
		"kube-node" => ["#{$instance_name_prefix}-0[1:#{$kube_node_instances}]"],
		"k8s-cluster:children" => ["kube-master", "kube-node"],
		"jump-host" => ["client-admin"]
	  }
	end
  end

end
end
end
