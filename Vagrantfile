# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

$install_dependencies = <<SCRIPT
yum makecache fast
yum install -y  python-virtualenv git gcc vim
#               python-devel openssl-devel libffi-devel libxml2-devel \
#               libxslt-devel
easy_install pip
pip install -U pip
pip install python-keystoneclient
SCRIPT

$install_keystone = <<SCRIPT
echo "export LC_ALL=en_US.UTF-8
export LANG=en_US.UTF-8" >> /etc/bashrc

mkdir -p /opt/stack
cd /opt/stack
git clone https://github.com/openstack/keystone.git
cd keystone
git checkout stable/juno
virtualenv .venv
source .venv/bin/activate
pip install -r requirements.txt
chown -R vagrant:vagrant /opt/stack/keystone
bin/keystone-manage db_sync
SCRIPT

$init_keystone = <<SCRIPT
cd /opt/stack/keystone
export OS_SERVICE_TOKEN=ADMIN
export OS_SERVICE_ENDPOINT=http://localhost:35357/v2.0
keystone tenant-create --name admin
keystone user-create --name admin --tenant admin --pass cisco123
keystone role-create --name admin
keystone user-role-add --user admin --role admin --tenant admin
keystone service-create  --name keystone --type identity --description "Keystone Identity Service"
service_id=$(keystone service-list | awk -F '| ' '/keystone/{print $2}')
keystone endpoint-create --service-id=$service_id --publicurl=http://10.2.0.2:5000/v2.0 --internalurl=http://10.2.0.2:5000/v2.0 --adminurl=http://10.2.0.2:35357/v2.0
unset OS_SERVICE_TOKEN
unset OS_SERVICE_ENDPOINT
keystone --os-username admin --os-password cisco123 --os-auth-url http://localhost:35357/v2.0 --os-tenant-name admin user-list
SCRIPT

$run_keystone = <<SCRIPT
cd /opt/stack/keystone
echo "current dir"
source .venv/bin/activate
nohup bin/keystone-all --config-file etc/keystone.conf.sample &
deactivate
SCRIPT


Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    config.vm.box = "chef/centos-7.0"
    config.vm.network "private_network", ip: "10.2.0.2"

    config.vm.provision "shell", inline: $install_dependencies
    config.vm.provision "shell", inline: $install_keystone
    config.vm.provision "shell", inline: $run_keystone
    config.vm.provision "shell", inline: $init_keystone
end
