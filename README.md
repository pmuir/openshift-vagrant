# openshift-vagrant

1. Install vagrant, virtualbox, cygwin. Add the following to the default for cygwin
   * rsync
   * ssh

2. Download RHEL 7.1 Vagrant box for VirtualBox from https://access.redhat.com/downloads/content/293/ver=1/rhel---7/1.0.1/x86_64/product-downloads
3. Donload the Red Hat Container Tools from  https://access.redhat.com/downloads/content/293/ver=1/rhel---7/1.0.1/x86_64/product-downloads
4. Unzip the cdk.zip file you downloaded in your home directory. This should create ~/cdk (/Users/username/cdk)
 
   ```
   $ cd
   $ unzip ~/Downloads/cdk-1.0-0.zip
   ```   
5. Install additional Vagrant plugins for using Red Hat Vagrant boxes. The installation of the first plugin make take several minutes Vagrant may install some additional gem files as needed.

   ```
   $ cd ~/cdk/plugins
   $ ls -1 \*.gem
   $ vagrant plugin install vagrant-registration-0.0.8.gem
   ```
   Verify the plugins are installed:
   
   ```
   $ vagrant plugin list
   ```
6. Add the RHEL Server boxes to Vagrant. This is the configured virtual machine image that you downloaded a previous step. You will be using this for running OpenShift Enterprise.

   ```
   $ vagrant box add --name rhel-server-virtualbox-7.1-3 ~/Downloads/rhel-server-virutalbox-7.1-3.x86_64.box
   ```
   Verify that the boxes got installed:
   
   ```
   $ vagrant box list
   ```
   The box image files will be stored in your home directory under ~/.vagrant.d. You will need adequate space there, approximately 2GB.
7. If you are using cygwin, then edit your .bashrc and add

   ```
   export VAGRANT_DETECTED_OS=CYGWIN_NT-6.3
   ```
   (see https://github.com/mitchellh/vagrant/issues/5930)
8. To register any box you run with Red Hat Subscription Manager, edit ~/.vagrant.d/Vagrantfile and add
   
   ```
   Vagrant.configure('2') do |config|
     config.registration.username = 'rhn-support-pmuir'
     config.registration.password = ''
   end
   ```
   Note you need to put your own username and password in here. 
   Make sure you use a subscription which has an OpenShift Enterprise 3 entitlement. If you are an employee, use an employee subscription - the usernames will start with `rhn-`
9. Run
   
   ```
   vagrant up
   ```



