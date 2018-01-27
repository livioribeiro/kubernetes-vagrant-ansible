# -*- mode: ruby -*-
# vi: set ft=ruby :

$internal_ip = '10.0.0.'
$external_ip = '192.168.1.'

$n_nodes = 2
$n_etcd = 3

Vagrant.configure("2") do |config|

  config.vm.box = "bento/centos-7.4"
  config.vm.provider "virtualbox" do |vb|
    vb.linked_clone = true
  end

  config.vm.define "squid" do |machine|
    machine.vm.network "private_network", ip: $internal_ip + '30'

    machine.vm.synced_folder ".cache/squid", "/var/spool/squid", # type: "rsync", rsync__chown: false
      owner: "root", group: "root", mount_options: ["dmode=777,fmode=777"]

    machine.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
    end
  end

  (1..$n_etcd).each do |i|
    config.vm.define "etcd-#{i}" do |machine|
      machine.vm.provider "virtualbox" do |vb|
        vb.memory = "1024"
      end

      machine.vm.hostname = "etcd-#{i}"
      machine.vm.network "private_network", ip: $internal_ip + (20 + i).to_s
    end
  end

  (1..$n_nodes).each do |i|
    config.vm.define "node-#{i}" do |machine|
      machine.vm.provider "virtualbox" do |vb|
        vb.memory = "2048"
      end

      machine.vm.hostname = "node-#{i}"

      machine.vm.network "private_network", ip: $internal_ip + (10 + i).to_s
      machine.vm.network "public_network", ip: $external_ip + (10 + i).to_s, bridge: "enp3s0"
    end
  end

  config.vm.define "master" do |machine|
    machine.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
    end

    machine.vm.hostname = "master"

    machine.vm.network "private_network", ip: $internal_ip + '10'
    machine.vm.network "public_network", ip: $external_ip + '10', bridge: "enp3s0"

    config.vm.provision "ansible" do |ansible|
      ansible.playbook = "provisioning/playbook.yaml"
      ansible.limit = "all"
      ansible.groups = {
        "nodes" => (1..$n_nodes).map { |i| "node-#{i}" },
        "etcd" => (1..$n_etcd).map { |i| "etcd-#{i}" }
      }
    end
  end

end
