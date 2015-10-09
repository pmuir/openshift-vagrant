# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure('2') do |config|
	# it is important that the name of this box is "default"
	config.vbguest.auto_update = false

	windows = (/cygwin|mswin|mingw|bccwin|wince|emx/ =~ RUBY_PLATFORM) != nil
    guest_additions = ENV['GUEST_ADDITIONS_PATH'] || (windows ? "C:\\Program files\\Oracle\\VirtualBox\\VBoxGuestAdditions.iso" :  "/opt/VirtualBox/VBoxGuestAdditions.iso")

	config.vm.provider "virtualbox" do |v|
		v.memory = 2048
		v.cpus = 2
		v.customize ["modifyvm", :id, "--natnet1", "172.12/16"]
		v.customize ["storageattach", :id, "--storagectl", "IDE Controller", "--port", "1", "--device", "0", "--type", "dvddrive", "--medium", guest_additions]
	end

	# Box name
	config.vm.hostname = 'openshift-enterprise-pmuir'
	config.vm.box = "rhel-server-virtualbox-7.1-3"
	config.vm.synced_folder '.', '/vagrant', disabled: true
	config.vm.synced_folder '.', '/mnt/vagrant', type: 'rsync'
	config.nfs.functional = false
	config.vm.network "private_network", ip: ENV['VM_IP'] || '10.0.2.15'

	# Attach subscriptions, enable repos, install packages
	# Fix this so we only attach one sub :-)
	config.vm.provision 'shell', inline: "subscription-manager list --installed | grep -q \"Red Hat OpenShift Enterprise\" || subscription-manager list --matches=\"*OpenShift Enterprise*\" --available --all --pool-only | subscription-manager attach --file=-"
	config.vm.provision 'shell', inline: "subscription-manager repos --enable=\"rhel-7-server-rpms\" --enable=\"rhel-7-server-extras-rpms\" --enable=\"rhel-7-server-ose-3.0-rpms\""
	config.vm.provision 'shell', inline: "rpm -qa | grep -q epel-release || rpm -ivh http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm"
	config.vm.provision 'shell', inline: "yum -y install wget git net-tools bind-utils iptables-services bridge-utils gcc python-virtualenv docker ansible && yum update -y && yum clean all"

	# Install Guest Additions
	config.vm.provision 'shell', inline: "mkdir /mnt/cdrom && mount /dev/cdrom /mnt/cdrom"
	config.vm.provision 'shell', inline: "/mnt/cdrom/VBoxLinuxAdditions.run"
	config.vm.provision 'shell', inline: "umount -f /mnt/cdrom && rm -rf /mnt/cdrom"
	config.vm.provision :reload

	# Configure Docker
	config.vm.provision 'shell', inline: "grep -q \"insecure-registry 172.30.0.0/16\" /etc/sysconfig/docker || echo \"INSECURE_REGISTRY='--selinux-enabled --insecure-registry 172.30.0.0/16'\" | tee --append /etc/sysconfig/docker > /dev/null"

	# Storage?

	# Configure ssh access
	config.vm.provision 'shell', inline: "rm -f /root/.ssh/id_rsa /root/.ssh/id_rsa.pub /root/.ssh/known_hosts"
	config.vm.provision 'shell', inline: "ssh-keygen -q -f /root/.ssh/id_rsa -N \"\" && cat  /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys && echo \"StrictHostKeyChecking=no\" >>  /root/.ssh/config && chmod 0600  /root/.ssh/authorized_keys  /root/.ssh/config && chown root:root /root/.ssh/*"

	config.vm.provision 'shell', inline: "systemctl stop docker > /dev/null 2>&1 || :" #in case this isn't first run
	config.vm.provision 'shell', inline: "groupadd docker > /dev/null 2>&1 || : "
	config.vm.provision 'shell', inline: "usermod -a -G docker vagrant"
	config.vm.provision 'shell', inline: "systemctl enable docker && systemctl start docker"
	config.vm.provision 'shell', inline: "chown root:docker /var/run/docker.sock"
	config.vm.provision 'shell', inline: "systemctl enable docker && systemctl start docker"

	# Run ansible
	config.vm.provision 'shell', inline: "rm -rf openshift-ansible && git clone https://github.com/openshift/openshift-ansible"
	config.vm.provision 'shell', inline: "mkdir -p /etc/ansible && cp /home/vagrant/sync/ansible/hosts /etc/ansible"
	config.vm.provision 'shell', inline: "ansible-playbook /home/vagrant/openshift-ansible/playbooks/byo/config.yml"

	# Uninstall Ansible
	config.vm.provision 'shell', inline: "yum -y remove ansible"

	# Cleanup
	config.vm.provision 'shell', inline: "yum -y autoremove && yum -y clean all"
end
