# Defines our Vagrant environment
#
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  # create ansible node
  config.vm.define :"ansible.local" do |ansible_config|
    ansible_config.vm.box = "centos/7"

    ansible_config.vm.hostname = "ansible.local"
    ansible_config.vm.define "ansible.local"
    ansible_config.vm.network :private_network, ip: "192.168.0.100", hostsupdater: "skip"
    ansible_config.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
	    vb.name = "ansible.local"

      vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
    end

    ansible_config.vm.synced_folder "../ansible-hortonworks", "/ansible-hortonworks"

    if Vagrant.has_plugin?("vagrant-vbguest")
      ansible_config.vbguest.auto_update = true
      ansible_config.vbguest.no_remote   = true
      ansible_config.vbguest.no_install  = false
    end

    if Vagrant.has_plugin?("vagrant-proxyconf")
      ansible_config.proxy.http     = ""
      ansible_config.proxy.https    = ""
      ansible_config.proxy.no_proxy = ""
    end

    if Vagrant.has_plugin?("vagrant-hosts")
      ansible_config.vm.provision :hosts do |provisioner|
        provisioner.sync_hosts = true
        provisioner.add_localhost_hostnames = false

        provisioner.add_host '192.168.0.100', ['ansible.local', 'ansible']
        provisioner.add_host '192.168.1.10',  ['master.local', 'master']
        (0..1).each do |i|
          provisioner.add_host "192.168.1.10#{i}", ["slave#{i}.local", "slave#{i}"]
        end
      end
    end

    ansible_config.vm.provision :shell, :inline  => '
    sudo rm -f ~/.ssh/id_rsa     >/dev/null 2>&1
    sudo rm -f ~/.ssh/id_rsa.pub >/dev/null 2>&1
    ', privileged: false

    ansible_config.vm.provision "file", source: "../ssh/id_rsa", destination: "~/.ssh/id_rsa"
    ansible_config.vm.provision "file", source: "../ssh/id_rsa.pub", destination: "~/.ssh/id_rsa.pub"

    ansible_config.vm.provision :shell, :inline  => '
    chmod 600 ~/.ssh/id_rsa
    chmod 600 ~/.ssh/id_rsa.pub
    ', privileged: false

    ansible_config.vm.provision :shell, :inline  => '
    sudo yum -y install epel-release || sudo yum -y install http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
    sudo yum -y install gcc gcc-c++ python-virtualenv python-pip python-devel libffi-devel openssl-devel libyaml-devel sshpass git vim-enhanced
    virtualenv ~/ansible
    source ~/ansible/bin/activate && pip install setuptools --upgrade
    source ~/ansible/bin/activate && pip install pip --upgrade
    source ~/ansible/bin/activate && pip install pycparser functools32 pytz ansible==2.3.2
    ', privileged: false

  end
end
