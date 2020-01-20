# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.define "machine1" do |machine1|
	  machine1.vm.box = "debian/buster64"
          machine1.vm.box_version = "10.0.0"
          machine1.vm.synced_folder ".", "/vagrant"
	  machine1.vm.provision "ansible" do |ansible|
		  ansible.playbook = "playbooks/playbook-machine1.yml"
                  ansible.extra_vars = { ansible_python_interpreter:"/usr/bin/python3" }
                  #ansible.provisioning_path = "/vagrant/playbooks"
	  end
          machine1.vm.provider "virtualbox" do |vb|
                  #vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
                  #vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
                  vb.memory = 4096
                  vb.cpus = 3
          end
          machine1.vm.network "private_network", ip: "10.0.0.2"
          machine1.vm.network "forwarded_port", guest: 31112, host: 31112, auto_correct: true
          machine1.vm.network "forwarded_port", guest: 31113, host: 31113, auto_correct: true
          machine1.vm.network "forwarded_port", guest: 31114, host: 31114, auto_correct: true
  end
end
