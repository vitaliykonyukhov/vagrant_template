# -*- mode: ruby -*-
# vi: set ft=ruby :
ENV['VAGRANT_EXPERIMENTAL'] = "disks"

require 'yaml'

current_dir = File.dirname(File.expand_path(__FILE__))
local_config = YAML.load_file("#{current_dir}/vagrant_vars.yml")

# *** CONFIG begin

# # Number of nodes
nb_nodes = local_config["nodes"]

# Ansible playbook loacation
ansible_playbook_common = local_config["ansible_playbook_common"]

# *** CONFIG end

# ** Define settings for every node from vars file
nb_nodes.each do |node_nb|
  Vagrant.configure("2") do |config|
    # ** Global config for all machines:
    config.vm.box = "#{node_nb["vm_box"]}"

    # Disable default rsync of current host directory
    config.vm.synced_folder ".", "/vagrant", disabled: true

    config.vm.define "#{node_nb["name"]}" do |machine|
      machine.vm.hostname = "#{node_nb["name"]}"

      if !node_nb["ip"].nil? && !node_nb["netmask"].nil?
        machine.vm.network "private_network", ip: "#{node_nb["ip"]}", netmask: "#{node_nb["netmask"]}"
      end
      if !node_nb["disk"].nil? 
        machine.vm.disk :disk, size: "#{node_nb["disk"]}", name: "disk-#{node_nb["name"]}"
      end
      machine.vm.provider "virtualbox" do |vb|
        vb.customize ["modifyvm", :id, "--name", "#{node_nb["name"]}"]
        vb.gui = false
        vb.memory = "#{node_nb["memory"]}"
        vb.cpus = "#{node_nb["cpu"]}"
      end

      # Ansible provisioning
      # Execute Ansible provisioner only when playbok is defined in vm config
      if !node_nb["ansible_playbook"].nil? && (node_nb["ansible_playbook"] != [])
        node_nb["ansible_playbook"].each do |pb_loc|
          machine.vm.provision :ansible do |ansible|
            ansible.playbook = pb_loc
            if !node_nb["ansible_raw_arguments"].nil? && (node_nb["ansible_raw_arguments"] != "")
              ansible.raw_arguments = node_nb["ansible_raw_arguments"]
            end
          end
        end
      end

      # Ansible provisioning
      # Execute Ansible provisioner only when all machines are defined
      # ie only for the last VM
      if node_nb["name"] == nb_nodes.last["name"]
        if !ansible_playbook_common.nil? && (ansible_playbook_common != [])
          ansible_playbook_common.each do |playbook_location|
            machine.vm.provision :ansible do |ansible|
              ansible.playbook = playbook_location
              if local_config["ansible_raw_arguments"] != ""
                ansible.raw_arguments = local_config["ansible_raw_arguments"]
              end
              # # Disable default limit to connect to all the machines
              ansible.limit = "all"
            end
          end
        end
      end
    end
  end
end