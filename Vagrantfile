# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  mesos_master_number_of_cluster = 3
  spark_number_of_cluster = 3
  config.vm.box = "ubuntu/trusty64"

  # mesos serverの設定
  (1..mesos_master_number_of_cluster).each do |machine_id|
    config.vm.define "mesos#{machine_id}" do |node|
      node.vm.hostname = "mesos#{machine_id}"
      node.vm.network "private_network", ip: "192.168.10.#{machine_id}", virtualbox_inet:"test_cluster"

      node.vm.provision :ansible do |ansible|
        # Disable default limit to connect to all the machines
        ansible.limit = "all"
        ansible.playbook = "mesos.yml"
      end

      node.vm.provider "virtualbox" do |vb|
        # memory and cpu
        vb.customize ["modifyvm", :id, "--memory", "1024", "--cpus", "2", "--ioapic", "on"]
        # Set the time to HOST
        vb.customize ["setextradata", :id, "VBoxInternal/Devices/VMMDev/0/Config/GetHostTimeDisabled", 0]
        # use host dns
        vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      end

    end
  end

  # spark serverの設定
  (1..spark_number_of_cluster).each do |machine_id|
    config.vm.define "spark#{machine_id}" do |node|
      node.vm.hostname = "spark#{machine_id}"
      node.vm.network "private_network", ip: "192.168.20.#{machine_id}", virtualbox_inet:"test_cluster"

      node.vm.provision :ansible do |ansible|
        # Disable default limit to connect to all the machines
        ansible.limit = "all"
        ansible.playbook = "spark.yml"
      end

      node.vm.provider "virtualbox" do |vb|
        # memory and cpu
        vb.customize ["modifyvm", :id, "--memory", "1024", "--cpus", "2", "--ioapic", "on"]
        # Set the time to HOST
        vb.customize ["setextradata", :id, "VBoxInternal/Devices/VMMDev/0/Config/GetHostTimeDisabled", 0]
        # use host dns
        vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      end

    end
  end

end
