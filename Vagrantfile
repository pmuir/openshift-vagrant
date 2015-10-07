# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure('2') do |config|
  # it is important that the name of this box is "default"
  config.vm.define :default do |vagrant_host|
    # Box name
    vagrant_host.vm.hostname = 'openshift-enterprise-pmuir'
    vagrant_host.vm.box = "rhel-server-virtualbox-7.1-3"
    vagrant_host.vm.synced_folder '.', '/vagrant', disabled: true
    vagrant_host.vm.synced_folder '.', '/mnt/vagrant', type: 'rsync'
    vagrant_host.nfs.functional = false
		
		
	vagrant_host.vm.provision 'shell', inline: "sudo subscription-manager list --installed | grep -q \"Red Hat OpenShift Enterprise\" || sudo subscription-manager list --matches=\"*OpenShift Enterprise*\" --available --all --pool-only | sudo subscription-manager attach --file=-"
    vagrant_host.vm.provision 'shell', inline: "sudo subscription-manager repos --enable=\"rhel-7-server-rpms\" --enable=\"rhel-7-server-extras-rpms\" --enable=\"rhel-7-server-ose-3.0-rpms\""
	vagrant_host.vm.provision 'shell', inline: "sudo yum -y install wget git net-tools bind-utils iptables-services bridge-utils gcc python-virtualenv docker"
    vagrant_host.vm.provision 'shell', inline: "sudo yum update -y" 
	vagrant_host.vm.provision 'shell', inline: "grep -q \"insecure-registry 172.30.0.0/16\" /etc/sysconfig/docker || echo \"INSECURE_REGISTRY='--selinux-enabled --insecure-registry 172.30.0.0/16'\" | sudo tee --append /etc/sysconfig/docker > /dev/null"

    # Storage?
	vagrant_host.vm.provision 'shell', inline: "rm -f /home/vagrant/.ssh/id_rsa /home/vagrant/.ssh/id_rsa.pub /home/vagrant/.ssh/known_hosts"
	vagrant_host.vm.provision 'shell', inline: "ssh-keygen -q -f /home/vagrant/.ssh/id_rsa -N \"\" && cat  /home/vagrant/.ssh/id_rsa.pub >> /home/vagrant/.ssh/authorized_keys && echo \"StrictHostKeyChecking=no\" >>  /home/vagrant/.ssh/config && chmod 0600  /home/vagrant/.ssh/authorized_keys  /home/vagrant/.ssh/config"
	
	
	vagrant_host.vm.provision 'shell', inline: "sudo systemctl stop docker > /dev/null 2>&1 || :" #in case this isn't first run
    vagrant_host.vm.provision 'shell', inline: "sudo groupadd docker > /dev/null 2>&1 || : "
    vagrant_host.vm.provision 'shell', inline: "sudo usermod -a -G docker vagrant"
    vagrant_host.vm.provision 'shell', inline: "sudo systemctl enable docker && sudo systemctl start docker"
    vagrant_host.vm.provision 'shell', inline: "sudo chown root:docker /var/run/docker.sock"
    vagrant_host.vm.provision 'shell', inline: "sudo systemctl enable docker && sudo systemctl start docker"
	
  end
end
