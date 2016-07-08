# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  mesos_master_number_of_cluster = 3
  mesos_master_ip_1to3_octed = "172.168.1."
  mesos_hostname_prefix = "mesos"
  mesos_master_zk_lists = []
  mesos_zoo_cfg_host = []
  (1..mesos_master_number_of_cluster).each do |machine_id|
    mesos_master_zk_lists << "#{mesos_master_ip_1to3_octed}#{10+machine_id}:2181"
    mesos_zoo_cfg_host << "server.#{machine_id}=#{mesos_master_ip_1to3_octed}#{10+machine_id}:2888:3888"
  end
  mesos_master_zk_url = "zk://" + mesos_master_zk_lists.join(",") + "/mesos"
  mesos_master_marathon_url = "zk://" + mesos_master_zk_lists.join(",") + "/marathon"
  mesos_zoo_cfg = mesos_zoo_cfg_host.join("\n")

  spark_number_of_cluster = 3
  config.vm.box = "ubuntu/trusty64"

  # mesos serverの設定
  (1..mesos_master_number_of_cluster).each do |machine_id|
    config.vm.define "#{mesos_hostname_prefix}#{machine_id}" do |node|
      node.vm.hostname = "mesos#{machine_id}"
      node.vm.network "private_network", ip: "#{mesos_master_ip_1to3_octed}#{10+machine_id}", virtualbox_inet:"test_cluster"

        node.vm.provision :ansible do |ansible|
          # Disable default limit to connect to all the machines
          ansible.limit = "all"
          ansible.playbook = "mesos.yml"
        end

       node.vm.provision :shell, :inline => "echo #{machine_id} | sudo tee /etc/zookeeper/conf/myid"
       node.vm.provision :shell, :inline => "echo #{mesos_master_zk_url} | sudo tee /etc/mesos/zk"
       node.vm.provision :shell, :inline => "echo -e '#{mesos_zoo_cfg}'"
       node.vm.provision :shell, :inline => "echo -e '#{mesos_zoo_cfg}' | sudo tee -a /etc/zookeeper/conf/zoo.cfg"
       node.vm.provision :shell, :inline => "echo #{(mesos_master_number_of_cluster.to_f/2).ceil} | sudo tee /etc/mesos-master/quorum"
       node.vm.provision :shell, :inline => "echo #{mesos_master_ip_1to3_octed}#{10+machine_id} | sudo tee /etc/mesos-master/ip"
       node.vm.provision :shell, :inline => "cp /etc/mesos-master/ip /etc/mesos-master/hostname"
       node.vm.provision :shell, :inline => "mkdir -p /etc/marathon/conf"
       node.vm.provision :shell, :inline => "cp /etc/mesos-master/ip /etc/marathon/conf/hostname"
       node.vm.provision :shell, :inline => "cp /etc/mesos-master/hostname /etc/marathon/conf"
       node.vm.provision :shell, :inline => "cp /etc/mesos/zk /etc/marathon/conf/master"
       node.vm.provision :shell, :inline => "cp /etc/marathon/conf/master /etc/marathon/conf/zk"
       node.vm.provision :shell, :inline => "echo debug start"
       node.vm.provision :shell, :inline => "echo #{mesos_master_marathon_url} | sudo tee /etc/marathon/conf/zk"
       node.vm.provision :shell, :inline => "echo debug end"
       node.vm.provision :shell, :inline => "echo manual | sudo tee /etc/init/mesos-slave.override"

       node.vm.provision :shell, :inline => "service zookeeper restart"
       node.vm.provision :shell, :inline => "service mesos-master restart"
       node.vm.provision :shell, :inline => "service marathon restart"

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
      node.vm.network "private_network", ip: "#{mesos_master_ip_1to3_octed}#{20+machine_id}", virtualbox_inet:"test_cluster"

        node.vm.provision :ansible do |ansible|
          # Disable default limit to connect to all the machines
          ansible.limit = "all"
          ansible.playbook = "spark.yml"
        end

      node.vm.provision :shell, :inline => "echo #{mesos_master_zk_url} | sudo tee /etc/mesos/zk"
      node.vm.provision :shell, :inline => "echo #{mesos_master_ip_1to3_octed}#{20+machine_id} | sudo tee /etc/mesos-slave/ip"
      node.vm.provision :shell, :inline => "echo #{mesos_master_ip_1to3_octed}#{20+machine_id} | sudo tee /etc/mesos-slave/hostname"
      node.vm.provision :shell, :inline => "echo manual | sudo tee /etc/init/zookeeper.override"
      node.vm.provision :shell, :inline => "echo manual | sudo tee /etc/init/mesos-master.override"
      node.vm.provision :shell, :inline => "echo manual | sudo tee /etc/init/marathon.override"
      node.vm.provision :shell, :inline => "echo manual | sudo tee /etc/init/chronos.override"
      node.vm.provision :shell, :inline => "service mesos-slave restart"

      node.vm.provider "virtualbox" do |vb|
        # memory and cpu
        vb.customize ["modifyvm", :id, "--memory", "512", "--cpus", "2", "--ioapic", "on"]
        # Set the time to HOST
        vb.customize ["setextradata", :id, "VBoxInternal/Devices/VMMDev/0/Config/GetHostTimeDisabled", 0]
        # use host dns
        vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      end

    end
  end

end
