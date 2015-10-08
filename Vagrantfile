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

    # Attach subscriptions, enable repos, install packages
    # Fix this so we only attach one sub :-)
    vagrant_host.vm.provision 'shell', inline: "subscription-manager list --installed | grep -q \"Red Hat OpenShift Enterprise\" || subscription-manager list --matches=\"*OpenShift Enterprise*\" --available --all --pool-only | subscription-manager attach --file=-"
    vagrant_host.vm.provision 'shell', inline: "subscription-manager repos --enable=\"rhel-7-server-rpms\" --enable=\"rhel-7-server-extras-rpms\" --enable=\"rhel-7-server-ose-3.0-rpms\""
    vagrant_host.vm.provision 'shell', inline: "rpm -qa | grep -q epel-release || rpm -ivh http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm"
    vagrant_host.vm.provision 'shell', inline: "yum -y install wget git net-tools bind-utils iptables-services bridge-utils gcc python-virtualenv docker ansible && yum update -y && yum clean all"

    # Configure Docker
    vagrant_host.vm.provision 'shell', inline: "grep -q \"insecure-registry 172.30.0.0/16\" /etc/sysconfig/docker || echo \"INSECURE_REGISTRY='--selinux-enabled --insecure-registry 172.30.0.0/16'\" | tee --append /etc/sysconfig/docker > /dev/null"

    # Storage?

    # Configure ssh access
    vagrant_host.vm.provision 'shell', inline: "rm -f /root/.ssh/id_rsa /root/.ssh/id_rsa.pub /root/.ssh/known_hosts"
    vagrant_host.vm.provision 'shell', inline: "ssh-keygen -q -f /root/.ssh/id_rsa -N \"\" && cat  /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys && echo \"StrictHostKeyChecking=no\" >>  /root/.ssh/config && chmod 0600  /root/.ssh/authorized_keys  /root/.ssh/config && chown root:root /root/.ssh/*"

    vagrant_host.vm.provision 'shell', inline: "systemctl stop docker > /dev/null 2>&1 || :" #in case this isn't first run
    vagrant_host.vm.provision 'shell', inline: "groupadd docker > /dev/null 2>&1 || : "
    vagrant_host.vm.provision 'shell', inline: "usermod -a -G docker vagrant"
    vagrant_host.vm.provision 'shell', inline: "systemctl enable docker && systemctl start docker"
    vagrant_host.vm.provision 'shell', inline: "chown root:docker /var/run/docker.sock"
    vagrant_host.vm.provision 'shell', inline: "systemctl enable docker && systemctl start docker"

    # Run ansible
    vagrant_host.vm.provision 'shell', inline: "rm -rf openshift-ansible && git clone https://github.com/openshift/openshift-ansible"
    vagrant_host.vm.provision 'shell', inline: "mkdir -p /etc/ansible && cp /home/vagrant/sync/ansible/hosts /etc/ansible"
    vagrant_host.vm.provision 'shell', inline: "ansible-playbook /home/vagrant/openshift-ansible/playbooks/byo/config.yml"

    # Uninstall Ansible
    vagrant_host.vm.provision 'shell', inline: "yum -y remove ansible"

    # Cleanup
    vagrant_host.vm.provision 'shell', inline: "yum -y autoremove && yum -y clean all"

  end
end
