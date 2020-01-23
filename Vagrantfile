# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.define "imnoserver" do |imnoserver|
	  imnoserver.vm.box = "debian/buster64"
          imnoserver.vm.box_version = "10.0.0"
          imnoserver.vm.synced_folder ".", "/vagrant"
	  imnoserver.vm.provision "ansible_local" do |ansible|
		  ansible.playbook = "playbook-imnoserver.yml"
                  ansible.extra_vars = { ansible_python_interpreter:"/usr/bin/python3" }
                  ansible.provisioning_path = "/vagrant/playbooks"
	  end
          imnoserver.vm.provider "virtualbox" do |vb|
                  #vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
                  #vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
                  vb.memory = 4096
                  vb.cpus = 3
          end
          imnoserver.vm.provider "libvirt" do |libvirt|
                  libvirt.cpus = 3
                  libvirt.memory = 4096
          end
          imnoserver.vm.network "private_network", ip: "10.0.0.2"
          imnoserver.vm.network "forwarded_port", guest: 31112, host: 31112, auto_correct: true
          imnoserver.vm.network "forwarded_port", guest: 31113, host: 31113, auto_correct: true
          imnoserver.vm.network "forwarded_port", guest: 31114, host: 31114, auto_correct: true
  end
end
