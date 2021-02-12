# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'
# Get developers username.
user = ENV['USER']
# Opens developer specific vagrant config file containing options
# for ports and sync directories.
vars = YAML.load_file "./vagrant.config/vagrant.#{user}.yml"

# Reused vars for DRY.
guest_sync_dir = '/var/www/node_app'
guest_port = '8090'
guest_db_port = '5432'

Vagrant.configure("2") do |config|
  config.vm.box = "geerlingguy/ubuntu2004"
  config.vm.network "forwarded_port", guest: guest_port,
    host: vars['hport']
  config.vm.network "forwarded_port", guest: guest_db_port,
    host: vars['dbhport']
  config.ssh.insert_key = false
  config.vm.synced_folder vars['hfolder'], guest_sync_dir

  config.vm.provider :virtualbox do |v|
    v.name = "cbrmApp-vm"
    v.memory = 1024
    v.cpus = 1
  end

  # Ansible provisioner.
  config.vm.provision "ansible" do |ansible|
    ansible.compatibility_mode="2.0"
    ansible.playbook = 'provisioning/playbook.yml'
    ansible.inventory_path = ".vagrant/provisioners/ansible/inventory/" +
                              "vagrant_ansible_inventory"
    ansible.host_vars = {
      "ansible_python_interpreter" => "/usr/bin/python3",
      "ansible_ssh_extra_args" => "-o StrictHostKeyChecking=no"
   }
    ansible.limit = "all"
    ansible.galaxy_role_file = "provisioning/requirements.yml"
    # ansible.galaxy_command = "sudo ansible-galaxy install --role-file=%{role_file} /
    # --roles-path=%{roles_path} --force"
    ansible.become = true
    ansible.extra_vars = {
      vagrant_ssh_keyfile: "~/.vagrant.d/insecure_private_key",
      host_port: vars['hport'],
      guest_port: guest_port,
      db_guest_port:  guest_db_port,
      node_app_location: guest_sync_dir
    }
    # Enables passing of args to Ansible from Vagrant CLI
    # via the ANSIBLE_ARGS environment variable
    if ENV["ANSIBLE_ARGS"]
      ansible.raw_arguments = Shellwords.shellsplit(ENV["ANSIBLE_ARGS"])
    end
  end
end
