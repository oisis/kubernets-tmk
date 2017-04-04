# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    config.vm.box_check_update = false
    # To create and import virtualbox box you need to run those commands:
    # git clone https://github.com/oisis/packer-centos7
    # cd ./packer-centos7
    # packer build -only=virtualbox-iso template.json
    # vagrant box add centos71 ./centos71-x64-virtualbox.box
    # If you already have box centos71 and would like to switch
    # to new one run this command: vagrant box remove centos71
    config.vm.box = "centos71"
    config.vm.provider :virtualbox do |vb|
        vb.gui = false
    end

$fill_hosts = <<SCRIPT
cat > /etc/hosts <<EOF
127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4
::1       localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.10.11 etcd1.vagrant.loc etcd1
192.168.10.12 etcd2.vagrant.loc etcd2
192.168.10.13 etcd3.vagrant.loc etcd3
192.168.10.21 k8s1.vagrant.loc k8s1
192.168.10.22 k8s2.vagrant.loc k8s2
192.168.10.31 node1.vagrant.loc node1
192.168.10.32 node2.vagrant.loc node2
192.168.10.100 lb.vagrant.loc lb
EOF
mkdir -p /home/vagrant/.ssh && chmod 0700 /home/vagrant/.ssh
VAGRANT_SSH_KEY=`grep vagrant /home/vagrant/.ssh/authorized_keys`
echo $VAGRANT_SSH_KEY > /home/vagrant/.ssh/authorized_keys.org
cat /home/vagrant/.ssh/authorized_keys.org > /home/vagrant/.ssh/authorized_keys
cat /vagrant/.ansible/id_rsa.pub >> /home/vagrant/.ssh/authorized_keys
rm -f /home/vagrant/.ssh/authorized_keys.org
/bin/cp /vagrant/.ansible/id* /home/vagrant/.ssh/
chown -R vagrant:vagrant /home/vagrant/.ssh/
SCRIPT

  (1..3).each do |i|
    config.vm.define "etcd#{i}" do |config|
      config.vm.hostname = "etcd#{i}.vagrant.loc"
      config.vm.provision "shell", inline: $fill_hosts
      config.vm.network :private_network,ip: "192.168.10.1#{i}"
      config.vm.provision :ansible do |ansible|
        ansible.verbose = "false"
        ansible.playbook = "playbook.yaml"
        ansible.limit = "etcd#{i}.vagrant.loc"
        ansible.inventory_path = "environments/vagrant/inventory"
        ansible.raw_arguments  = [ "--connection=paramiko", "-e env=vagrant", "--vault-password-file=./.ansible/ansible_password" ]
      end
      config.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", "256", "--cpus", 1, "--ioapic", "on", "--cpuexecutioncap", "50"]
      end
    end
  end

  (1..2).each do |i|
    config.vm.define "k8s#{i}" do |config|
      config.vm.hostname = "k8s#{i}.vagrant.loc"
      config.vm.provision "shell", inline: $fill_hosts
      config.vm.network :private_network,ip: "192.168.10.2#{i}"
      config.vm.network :forwarded_port, guest: "8080", host: "808#{i}"
      config.vm.provision :ansible do |ansible|
        ansible.verbose = "false"
        ansible.playbook = "playbook.yaml"
        ansible.limit = "k8s#{i}.vagrant.loc"
        ansible.inventory_path = "environments/vagrant/inventory"
        ansible.raw_arguments  = [ "--connection=paramiko", "-e env=vagrant", "--vault-password-file=./.ansible/ansible_password" ]
      end
      config.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", "512", "--cpus", 1, "--ioapic", "on", "--cpuexecutioncap", "50"]
      end
    end
  end

  (1..2).each do |i|
    config.vm.define "node#{i}" do |config|
      config.vm.hostname = "node#{i}.vagrant.loc"
      config.vm.provision "shell", inline: $fill_hosts
      config.vm.network :private_network,ip: "192.168.10.3#{i}"
      config.vm.provision :ansible do |ansible|
        ansible.verbose = "false"
        ansible.playbook = "playbook.yaml"
        ansible.limit = "node1.vagrant.loc"
        ansible.inventory_path = "environments/vagrant/inventory"
        ansible.raw_arguments  = [ "--connection=paramiko", "-e env=vagrant", "--vault-password-file=./.ansible/ansible_password" ]
      end
      config.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", "1024", "--cpus", 1, "--ioapic", "on", "--cpuexecutioncap", "50"]
      end
    end
  end

  config.vm.define "lb" do |config|
    config.vm.hostname = "lb.vagrant.loc"
    config.vm.provision "shell", inline: $fill_hosts
    config.vm.network :private_network,ip: "192.168.10.100"
    config.vm.network :forwarded_port, guest: 1936, host: 1936
    config.vm.network :forwarded_port, guest: 2379, host: 2379
    config.vm.network :forwarded_port, guest: 4001, host: 4001
    config.vm.network :forwarded_port, guest: 8080, host: 8080
    config.vm.network :forwarded_port, guest: 6443, host: 6443
    config.vm.provision :ansible do |ansible|
      ansible.verbose = "false"
      ansible.playbook = "playbook.yaml"
      ansible.limit = "lb.vagrant.loc"
      ansible.inventory_path = "environments/vagrant/inventory"
      ansible.raw_arguments  = [ "--connection=paramiko", "-e env=vagrant", "--vault-password-file=./.ansible/ansible_password" ]
    end
    config.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "512", "--cpus", 1, "--ioapic", "on", "--cpuexecutioncap", "50"]
    end
  end
end
