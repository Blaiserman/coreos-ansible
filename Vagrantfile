# -*- mode: ruby -*-
# # vi: set ft=ruby :

calico_node_ver = "master"

# Size of the cluster created by Vagrant
num_instances=1

# Change basename of the VM
instance_name_prefix="coreos"

# Official CoreOS channel from which updates should be downloaded
update_channel='stable'

Vagrant.configure("2") do |config|
  # always use Vagrants insecure key
  config.ssh.insert_key = false

  config.vm.box = "coreos-%s" % update_channel
  config.vm.box_version = ">= 308.0.1"
  config.vm.box_url = "http://%s.release.core-os.net/amd64-usr/current/coreos_production_vagrant.json" % update_channel
  
  # run from Linux sunsystem for linux
  config.ssh.insert_key = false
  config.ssh.private_key_path = "~/.vagrant.d/insecure_private_key"

  config.vm.provider :virtualbox do |v|
    # On VirtualBox, we don't have guest additions or a functional vboxsf
    # in CoreOS Container Linux, so tell Vagrant that so it can be smarter.
    v.check_guest_additions = false
    v.functional_vboxsf     = false
	(1..num_instances).each do |i|
	  v.name = "#{instance_name_prefix}-#{i}"
	  if i == 1
	    v.memory = 2048
	  else
	    v.memory = 1024
	  end
	end
  end

  # plugin conflict
  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  # Set up each box
  (1..num_instances).each do |i|
    vm_name = "%s-%02d" % [instance_name_prefix, i]
    config.vm.define vm_name do |host|
      host.vm.hostname = vm_name

      ip = "172.17.8.#{i+100}"
      host.vm.network :private_network, ip: ip
      # Workaround VirtualBox issue where eth1 has 2 IP Addresses at startup
      host.vm.provision :shell, :inline => "sudo /usr/bin/ip addr flush dev eth1"
      host.vm.provision :shell, :inline => "sudo /usr/bin/ip addr add #{ip}/24 dev eth1"

      # Pre-load the calico/node image.  This slows down the vagrant up
      # command, but speeds up the actual tutorial.
      # host.vm.provision :docker, images: ["quay.io/calico/node:#{calico_node_ver}", "busybox:latest"]

      # Use a different cloud-init on the first server.
      #if i == 1
        host.vm.provision :file, :source => "user-data-first", :destination => "/tmp/vagrantfile-user-data"
        host.vm.provision :shell, :inline => "mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/", :privileged => true
		# install python for ansible
		host.vm.provision :shell, :inline => "wget -qO- https://raw.githubusercontent.com/judexzhu/Install-Python-on-CoreOs/master/install-python.sh | sudo bash", :privileged => true
      # else
        # host.vm.provision :file, :source => "user-data-others", :destination => "/tmp/vagrantfile-user-data"
        # host.vm.provision :shell, :inline => "mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/", :privileged => true
      # end

      config.vm.post_up_message = "Vagrant has finished but cloud-init might still be executing.
      Check the progress using systemctl status -f"
	  
	  # ansible.groups = ansible_groups
    end
  end
end

