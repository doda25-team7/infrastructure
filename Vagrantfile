# -*- mode: ruby -*-
# vi: set ft=ruby :

NUM_WORKERS = 2

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-24.04"

  # Control node
  config.vm.define "ctrl" do |ctrl|
    ctrl.vm.hostname = "ctrl"
    ctrl.vm.network "private_network", ip: "192.168.56.100"
    
    ctrl.vm.provider "virtualbox" do |vb|
      vb.memory = "4096"
      vb.cpus = 1
    end

    # Ansible provisioning for control node
    ctrl.vm.provision "ansible_local" do |ansible|
      ansible.playbook = "ansible/general.yaml"
    end

    ctrl.vm.provision "ansible_local" do |ansible|
      ansible.playbook = "ansible/ctrl.yaml"
    end
  end

  # Worker nodes 
  (1..NUM_WORKERS).each do |i|
    config.vm.define "node-#{i}" do |node|
      node.vm.hostname = "node-#{i}"
      node.vm.network "private_network", ip: "192.168.56.#{100 + i}"
      
      node.vm.provider "virtualbox" do |vb|
        vb.memory = "6144"
        vb.cpus = 2
      end

      # Ansible provisioning for worker nodes
      node.vm.provision "ansible_local" do |ansible|
        ansible.playbook = "ansible/general.yaml"
      end

      node.vm.provision "ansible_local" do |ansible|
        ansible.playbook = "ansible/node.yaml"
      end
    end
  end
end
