# OCP3.11-AIO-IBM-Cloud-VSI

Intallin OCP 3.11 AIO on IBM Cloud

System requirements to configure cluster.
* Master node has up to 16G memory and up to 4 vCPU.
* Compute node has up to 8G memory and up to 1 vCPU.
* On all nodes, base OS is RHEL(CentOS) 7.4 or later (this example is based on CentOS 7.5)

In reason of file system been ext2 the DOCKER_STORAGE_OPTIONS  needs be changed  to work with ext file system  
sudo vi /etc/sysconfig/docker-storage
From:
DOCKER_STORAGE_OPTIONS="--overlay2"

To:
DOCKER_STORAGE_OPTIONS="--storage-driver devicemapper"


On All Nodes, Create a user for installation to be used in Ansible and also grant root privileges to him

 [root@master ~]# useradd origin

[root@master ~]# passwd origin

[root@master ~]# echo -e 'Defaults:origin !requiretty\norigin ALL = (root) NOPASSWD:ALL' | tee /etc/sudoers.d/openshift

[root@master ~]# chmod 440 /etc/sudoers.d/openshift

# if Firewalld is running, allow SSH

[root@master ~]# firewall-cmd --add-service=ssh --permanent

[root@master ~]# firewall-cmd --reload 
 
On All Nodes, install OpenShift Origin 3.11 repository and Docker and so on.

[root@master ~]# yum -y install centos-release-openshift-origin311 epel-release docker git pyOpenSSL
[root@master ~]# systemctl start docker

[root@master ~]# systemctl enable docker 



On server, install OpenShift Origin 3.11 repository and Docker and so on. 
[origin@master ~]$ ssh-keygen -q -N ""

Enter file in which to save the key (/home/origin/.ssh/id_rsa):
[origin@master ~]$ vi ~/.ssh/config
# create new ( define each node )

Host master
    Hostname master.example.net
    User origin

[origin@master ~]$ chmod 600 ~/.ssh/config
# transfer public-key to other nodes

[origin@master ~]$ ssh-copy-id master

origin@node01.srv.world's password:


Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'master'" and check to make sure that only the key(s) you wanted were added.

[origin@master ~]$ ssh-copy-id master


 On Master, login with a user created above and run Ansible Playbook for setting up OpenShift Cluster.

[origin@master ~]$ sudo yum -y install openshift-ansible 

 [origin@master ~]$ sudo vi /etc/ansible/hosts
[OSEv3:children]
masters
nodes
etcd

[OSEv3:vars]
# admin user created in previous section
ansible_ssh_user=origin
ansible_become=true
openshift_deployment_type=origin
openshift_disable_check=docker_storage

# use HTPasswd for authentication
openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider'}]

# define default sub-domain for Master node
openshift_master_default_subdomain=apps.rcleme170379.net

# allow unencrypted connection within cluster
openshift_docker_insecure_registries=172.30.0.0/16

[masters]
masterokd.example.net hostname="masterokd.example.net" public_hostname="masterokd.example.net" openshift_public_hostname="masterokd.example.net"

[etcd]
masterokd.example.net

[nodes]
masterokd.example.net openshift_node_group_name='node-config-all-in-one'

# if you'd like to separate Master node feature and Infra node feature, set like follows
# master.example.net openshift_node_group_name='node-config-master'
# node01.srv.world openshift_node_group_name='node-config-compute'
# node02.srv.world openshift_node_group_name='node-config-infra'

# run Prerequisites Playbook
[origin@masterokd ~]$ 

[origin@master ~]$ ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/prerequisites.yml 


origin@master ~]$ 
ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/prerequisites.yml 

................
................

PLAY RECAP *********************************************************************
master.example.net             : ok=83   changed=22   unreachable=0    failed=0



INSTALLER STATUS ***************************************************************
Initialization  : Complete (0:01:18)

# run Deploy Cluster Playbook

[origin@master ~]$ ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/deploy_cluster.yml 






...............
................

PLAY RECAP *********************************************************************
master.example.net            : ok=711  changed=322  unreachable=0    failed=0



INSTALLER STATUS ***************************************************************
Initialization               : Complete (0:00:39)
Health Check                 : Complete (0:00:56)
Node Bootstrap Preparation   : Complete (0:03:47)
etcd Install                 : Complete (0:01:42)
Master Install               : Complete (0:07:06)
Master Additional Install    : Complete (0:01:04)
Node Join                    : Complete (0:00:20)
Hosted Install               : Complete (0:01:27)
Cluster Monitoring Operator  : Complete (0:01:25)
Web Console Install          : Complete (0:00:41)
Console Install              : Complete (0:00:30)
metrics-server Install       : Complete (0:00:01)
Service Catalog Install      : Complete (0:02:40)
# show state

[origin@master ~]$ oc get nodes 

[origin@masterokd ~]$ oc get nodes 
NAME                         STATUS    ROLES                  AGE       VERSION
masterokd.example.net   Ready     compute,infra,master   3d        v1.11.0+d4cacc0
[origin@masterokd ~]$ 
