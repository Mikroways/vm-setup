# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"

  config.vm.provider "virtualbox" do |vb|
     vb.memory = "4096"
  end
    config.vm.provision :ansible do |ansible|
        ansible.galaxy_role_file = "ansible/requirements/roles.yml"
        ansible.playbook = "ansible/playbooks/vm-setup.yml"
        ansible.raw_ssh_args = ["-o ForwardAgent=yes"]
        ansible.extra_vars = {
          ansible_user: "vagrant",
          ansible_python_interpreter: "/usr/bin/python3",
          mw_tools_enabled: false
        }
    end
end
