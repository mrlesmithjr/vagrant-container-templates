# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.

# Ensure yaml module is loaded
require 'yaml'

# Read yaml container definitions to create
# **Update containers.yml to reflect any changes
containers = YAML.load_file(File.join(File.dirname(__FILE__), 'containers.yaml'))

# Define global variables
#
ENV['VAGRANT_DEFAULT_PROVIDER'] = 'docker'

Vagrant.configure(2) do |config|
  # Iterate over containers to get a count
  # Define as 0 for counting the number of containers to create from containers.yml
  groups = [] # Define array to hold ansible groups
  num_containers = 0
  populated_ansible_groups = Hash.new # Create hash to contain iterated groups

  # Create array of Ansible Groups from iterated containers
  containers.each do |container|
    num_containers = container
    container['ansible_groups'].each do |group|
      groups.push(group)
    end
  end

  # Remove duplicate Ansible Groups
  groups = groups.uniq

  # Iterate through array of Ansible Groups
  groups.each do |group|
    group_containers = []
    # Iterate list of containers
    containers.each do |container|
      container['ansible_groups'].each do |containergroup|
        # Check if container is a member of iterated group
        if containergroup == group
          group_containers.push(container['name'])
        end
      end
      populated_ansible_groups[group] = group_containers
    end
  end

  # Dynamic Ansible Groups iterated from containers.yml
  ansible_groups = populated_ansible_groups

  # config.ssh.username = 'root'
  # config.ssh.password = 'root'
  containers.each do |container_id|
    # Below is needed if not using Guest Additions
    # config.vm.synced_folder ".", "/vagrant", type: "rsync",
    #   rsync__exclude: "hosts"
    config.vm.define container_id['name'] do |container|
      container.vm.hostname = container_id['name']
      container.vm.provider "docker" do |d|
        if container_id['build']
          d.build_dir = "."
          d.has_ssh = true
          d.remains_running = true
        end
        if not container_id['build']
          d.image = container_id['image']
          d.remains_running = false
        end
      end

      # Port Forwards
      if not container_id['port_forwards'].nil?
        container_id['port_forwards'].each do |pf|
          container.vm.network :forwarded_port, \
          guest: pf['guest'], \
          host: pf['host']
        end
      end

      # Provisioners
      if not container_id['provision'].nil?
        if container_id['provision']
          #runs initial shell script
          # config.vm.provision :shell, path: "bootstrap.sh", keep_color: "true"
          if container_id == num_containers
            # container.vm.provision "ansible" do |ansible|
            #   ansible.limit = "all"
            #   #runs bootstrap Ansible playbook
            #   ansible.playbook = "bootstrap.yml"
            # end
            container.vm.provision "ansible" do |ansible|
              ansible.limit = "all"
              #runs Ansible playbook for installing roles/executing tasks
              ansible.playbook = "playbook.yml"
              ansible.groups = ansible_groups
            end
          end
        end
      end
    end
  end
end
