# -*- mode: ruby -*-
# vi: set ft=ruby :

$script_subscription_setup = <<SCRIPT
# Attach subscriptions, enable repos, install packages
# Fix this so we only attach one sub :-)
subscription-manager list --installed | grep -q \"Red Hat OpenShift Enterprise\" || subscription-manager list --matches=\"*OpenShift Enterprise*\" --available --all --pool-only | subscription-manager attach --file=-
subscription-manager repos --enable=\"rhel-7-server-rpms\" --enable=\"rhel-7-server-extras-rpms\" --enable=\"rhel-7-server-ose-3.0-rpms\"
rpm -qa | grep -q epel-release || rpm -ivh http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum -y install wget git net-tools bind-utils iptables-services bridge-utils gcc python-virtualenv docker ansible && yum update -y && yum clean all
SCRIPT

$script_configure_docker = <<SCRIPT
# Configure Docker
grep -q \"insecure-registry 172.30.0.0/16\" /etc/sysconfig/docker || echo \"INSECURE_REGISTRY='--selinux-enabled --insecure-registry 172.30.0.0/16'\" | tee --append /etc/sysconfig/docker > /dev/null

systemctl stop docker > /dev/null 2>&1 || :
groupadd docker > /dev/null 2>&1 || :
usermod -a -G docker vagrant
systemctl enable docker && systemctl start docker
chown root:docker /var/run/docker.sock
systemctl enable docker && systemctl start docker
SCRIPT

$script_run_ansible = <<SCRIPT
# Configure ssh access for Ansible
rm -f /root/.ssh/id_rsa /root/.ssh/id_rsa.pub /root/.ssh/known_hosts
ssh-keygen -q -f /root/.ssh/id_rsa -N \"\" && cat  /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys && echo \"StrictHostKeyChecking=no\" >>  /root/.ssh/config && chmod 0600  /root/.ssh/authorized_keys  /root/.ssh/config && chown root:root /root/.ssh/*

# Run Ansible
rm -rf openshift-ansible && git clone https://github.com/openshift/openshift-ansible
mkdir -p /etc/ansible && cp /home/vagrant/sync/ansible/hosts /etc/ansible
ansible-playbook /home/vagrant/openshift-ansible/playbooks/byo/config.yml
SCRIPT

$script_configure_openshift = <<SCRIPT
function stop_service {
  if [ "`systemctl is-active $1`" == "active" ]
  then
      echo "Service $1 was running. Stoppig it..."
      systemctl stop $1
      echo "$1 stopped"
  fi
}

# Comment out the docker network options for openshift-sdn.
# Not sure whether this is ok, but with these settings networking within the
# docker containers does not work properly and network operations will time out
# TODO - find out what the openshift-sdn does
if [ -f /run/openshift-sdn/docker-network ]; then
  echo "Disabling openshift-sdn Docker configuration options"
  sed -e '/DOCKER_NETWORK_OPTIONS/ s/^#*/#/' -i /run/openshift-sdn/docker-network
  systemctl restart docker
fi

# Stop the systemctl controlled Openshift services
# With these two services on the same machine, the master cannot find/detect
# the node
# TODO - Investigate the problem with this approach
stop_service openshift-sdn-node.service
stop_service openshift-sdn-master.service
stop_service openshift-node.service
stop_service openshift-master.service

# Start an all-in-one node
# TODO - This should be made a proper shell or even systemctl script
echo "Starting Openshift all-in-one"
openshift start --cors-allowed-origins=.* --public-master=https://localhost:8443 --volume-dir=/tmp/oo &> openshift.log &

# Give the server some time to come up
for i in {1..5}
do
  curl -ksSf https://10.0.2.15:8443/api
  if [ $? -ne 0 ]
  then
    echo "Waiting for OpenShift sever to come up"
    sleep 5
  else
    break
  fi
done

curl -ksSf https://10.0.2.15:8443/api
if [ $? -ne 0 ]
then
  echo "OpenShift startup seemed to have failed."
  exit -1
fi

# Create the Openshift registry
CREDENTIALS=/home/vagrant/openshift.local.config/master/openshift-registry.kubeconfig
KUBECONFIG=/home/vagrant/openshift.local.config/master/admin.kubeconfig
TEST=`oadm registry --create --dry-run --credentials=$CREDENTIALS --config=$KUBECONFIG`
if ! [[ "$TEST" =~ "service exists" ]]
then
  echo "Creating Openshift registry"
  oadm registry --create --credentials=$CREDENTIALS --config=$KUBECONFIG
fi

# TODO - pre-load some more image streams, templates, etc.
# Can be found in /home/vagrant/openshift-ansible/roles/openshift_examples/files/examples/
# Use 'oc create -f'
SCRIPT

$script_cleanup = <<SCRIPT
# Cleanup
yum -y remove ansible
yum -y autoremove && yum -y clean all
SCRIPT

Vagrant.configure('2') do |config|

    config.vbguest.auto_update = false

	config.vm.provider "virtualbox" do |v|
		v.memory = 2048
		v.cpus = 2
	end

	# Basic vm settings
	config.vm.hostname = 'openshift-enterprise'
	config.vm.box = "rhel-server-virtualbox-7.1-3"
	config.vm.synced_folder '.', '/vagrant', disabled: true
	config.nfs.functional = false

    # Configure networking. We will run the Openshift master so that it runs
    # on localhost in the guest which in turn allows us to just forward
    # Openshift relevant port
    # TODO - check which ports we really need
    config.vm.network "forwarded_port", guest: 80, host: 1080
    config.vm.network "forwarded_port", guest: 443, host: 1443
    config.vm.network "forwarded_port", guest: 5000, host: 5000
    config.vm.network "forwarded_port", guest: 8080, host: 8080
    config.vm.network "forwarded_port", guest: 8443, host: 8443

    # Provision the vm by running some shell scripts
    config.vm.provision "shell", inline: $script_subscription_setup
    config.vm.provision "shell", inline: $script_configure_docker
    config.vm.provision "shell", inline: $script_run_ansible
    config.vm.provision "shell", inline: $script_configure_openshift
    config.vm.provision "shell", inline: $script_cleanup
end
