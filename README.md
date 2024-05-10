# Adding-NewServer-OpenStack-AlmaLinux9
Adding New Compute Node with Existing OpenStack Cloud on AlmaLinux 9


 
 ## How to Add a New Compute Node with Existing OpenStack Cloud
 



### Pre-Installation for Adding New Compute Node With Existing OpenStack Cloud


#### This Configuration do on Compute Node

   #### Verifying the System Informations
#### Verifying Host Name
    hostnamectl
<!-- #### Verifying OS Version
    cat /etc/redhat-release -->
#### Verifying vmx Capable Processor
    grep -E ' svm | vmx' /proc/cpuinfo
#### Verifying kmv Enable in Processor
    lsmod | grep kvm
#### Verifying Processor Specification
    lscpu
#### Verifying RAM Usage
    free -h
#### Verifying Storage Partitions 
    lsblk

### Now Working On New Computer/Server, Which we want to connect with Existing OpenStack Cloud
    hostnamectl set-hostname cloud1
####
    ip a
####
    yum install nano -y

#### Need to Configure Static IP

#### Check Network Card Info file

    ls /etc/sysconfig/network-scripts/

##### Note: If OS are Almalinux 9 may be couldn't find NIC file in network-scripts folder. 
##### This problem for solutions:

#### Enable CRB  Repository

    dnf config-manager --set-enabled crb

#### Install Epel and Epel Next on Almalinux 9

    dnf install epel-release -y

#### Install OpenStack-PackStack Package, Here i'm choosing openstack-yoga Version 

    dnf install centos-release-openstack-yoga -y
#### YUM Packages Cache Clean
    yum clean all
#### Install network-scripts package
    yum install network-scripts -y
#### Enable/Start Network Service
    systemctl status network
    systemctl start network
    systemctl enable network
####
    systemctl restart network

#### Check Network Card Info file

    ls /etc/sysconfig/network-scripts/

#### Remember the IP and MAC Addresses
    ip a

#### Edit Network Interface File
    nano /etc/sysconfig/network-scripts/ifcfg-enp1s0

#### Paste bellow in ifcfg-enp1s0 file

    HWADRR=1c:1b:0d:8b:c6:ba
    NM_CONTROLLED=no
    BOOTPROTO=static
    ONBOOT=yes
    IPADDR=192.168.0.150
    PREFIX=24
    GATEWAY=192.168.0.1
    DNS1=8.8.8.8
    DNS2=8.8.4.4
    DEVICE=enp1s0
#### Connection UP Network Interface
    nmcli connection up enp1s0
#### Verifying Network Interface 
    ip a s enp1s0
####
    systemctl restart NetworkManager
####
    ping google.com
####
    cp /etc/sysconfig/network-scripts/ifcfg-enp1s0 /etc/sysconfig/network-scripts/ifcfg-enp1s0.bak
####
    cat /etc/sysconfig/network-scripts/ifcfg-enp1s0
####
#### Edit Hosts file:
#### For OpenStack All in One Controller Node

    echo "192.168.0.50 cloud.paulco.xyz cloud" >> /etc/hosts

#### For OpenStack New Compute Node

    echo "192.168.0.51 cloud1.paulco.xyz cloud1" >> /etc/hosts

#### Checking SELinux

    getenforce

#### Disable SELinux
    sed -i 's/^SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config
#### Disable Firewalld
    systemctl disable firewalld
    systemctl stop firewalld

#### Disable/Stop NetworkManager
    systemctl status NetworkManager
    systemctl disable NetworkManager
    systemctl stop NetworkManager
#### Network Interface UP
    ifup enp0s3
####
    yum autoremove epel-release
####
    yum autoremove openstack-packstack
#### 
    yum clean all
####
    yum repolist
####
    yum update -y && yum upgrade -y
####
    reboot
    
#### After Reboot check iptables status
    systemctl status iptables

### Now Working On OpenStack Controller Node
#### Go to OpenStack Controller node
#### Edit Hosts file:
#### For OpenStack Controller Node
    echo "192.168.0.50 cloud.paulco.xyz cloud" >> /etc/hosts
    
#### For OpenStack Compute Node
    echo "192.168.0.51 cloud1.paulco.xyz cloud1" >> /etc/hosts
####
#### SSH passwordless connection

    ssh-keygen
####
    cat ~/.ssh/id_rsa.pub
####
    ssh-copy-id root@192.168.0.95
####
    ssh root@192.168.0.95

#### From OpenStack Controller Node
#### 
    vi /etc/sysconfig/iptables
####
    iptables -L

#### Allow bellow mentioned Port in iptables roles

    iptables -I INPUT -p tcp --dport 5672 -j ACCEPT
    iptables -I INPUT -p tcp --dport 15672 -j ACCEPT
    iptables -I INPUT -p tcp --dport 3306 -j ACCEPT
    iptables -I INPUT -p udp --dport 53 -j ACCEPT
    iptables -I INPUT -p udp --dport 67 -j ACCEPT
    iptables -I INPUT -p tcp --dport 3260 -j ACCEPT
    #iptables -A INPUT -p tcp --dport 22 -s 0/0 -j ACCEPT

####
    service iptables save
####
    systemctl restart iptables

#### From OpenStack Controller Node
#### Edit the answer file

    cp answers.txt answers.txt.orginal
####
    vi answers.txt
#### Enter Existing Server IP or Hostname. 
    EXCLUDE_SERVERS= 192.168.0.50
#### Enter New Server IP or Hostname
    CONFIG_COMPUTE_HOSTS= 192.168.0.95
#### Setup NTP Server, Find NTP Section
    0.asia.pool.ntp.org,1.asia.pool.ntp.org,2.asia.pool.ntp.org,3.asia.pool.ntp.org

#### Before run bellow command, '#' will be removed.

    packstack --answer-file #/root/answers.txt | tee adding-Node-log.txt

<details>
 <summary> If You Get Error </summary>

#### Types of Error
Error: Pre installing Puppet and discovering hosts' details[ ERROR ] 
Error: GPG Keys are configured as: file:///etc/pki/rpm-gpg/RPM-GPG-KEY-AlmaLinux

#### Causes
Almalinux 8 Update & Upgrade related problem. we need to changes AlmaLinux 8 GPG key.

#### Solution:
Reference: https://almalinux.org/blog/2023-12-20-almalinux-8-key-update/ 
#### Import the the GPG key in almalinux8 
    rpm --import https://repo.almalinux.org/almalinux/RPM-GPG-KEY-AlmaLinux 
#### Clean Packages
    dnf clean packages
#### Now, Upgrade Almalinux-Release 
    dnf upgrade almalinux-release
</details>

#### When, Installation is Completed Verifying to the Compute Node

    rpm -qa | grep -i openstack

#### Check nova-compute log From Compute Node
    tail -f /var/log/nova/nova-compute.log
#### Install OpenStack Utilits
    yum install -y openstack-utils
#### Verifying OpenStack Services
    openstack-service status
#### Start OpenStack Services
    openstack-service start
#### Restart OpenStack Services
    openstack-service restart
#### Another Command, Verifying OpenStack Status
    openstack-status
#### Verifying to Open Virtual Switch
    ovs-vsctl show
#### Verifying on to Controller Node
    nova hypervisor-list
    
#### Add the compute node to the cell database
#### Important

#### Run the following commands on the controller node.

Source the admin credentials to enable admin-only CLI commands 

    . admin-openrc

#### Then confirm there are compute hosts in the database

    openstack compute service list --service nova-compute

#### Discover compute hosts:

    su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova

#### Alternatively, you can set an appropriate interval in /etc/nova/nova.conf:

    vi /etc/nova/nova.conf
#### Find **scheduler** Section in /etc/nova/nova.conf file
    [scheduler]
    discover_hosts_in_cells_interval = 300
#### Verifying to Compute Service List
    openstack compute service list
#### Verifying to Nova Status
    nova-status upgrade check
#### Verifying to Running VMs State
    virsh list --all
#### Setup New Server with New Server Hostname
    nova hypervisor-servers cloud1
#### Restart, Some OpenStack Services 
    systemctl restart openvswitch libvirtd neutron-openvswitch-agent openstack-nova-compute

<details>
 <summary> Error: IPtables OR IP Can't Access Related Errror </summary>

 #### Solution:
 Allow Your IP Block/CIDR in IPtables file
 ####
     sed -i.bak -e 's/\/32/\/16/' /etc/sysconfig/iptables
 Verifying IPtables lists
 ####
     ls -l /etc/sysconfig/iptables*
Restart IPtables Service
####
    systemctl restart iptables
Verifying IPtables, Another Approach 
####
    iptables -L

</details>

<details>
<summary> Error: Host is not mapped to any cell </summary>
Error: OpenStack error: Host is not mapped to any cell
 > Reference:
 > https://cloud.tencent.com/developer/article/1501368

#### Solution:
Go to Controler Node
#### 
    . keysourcerc_admin
####
    nova-manage cell_v2 discover_hosts --verbose
    
We can identify this using the openstack compute service list command:
#### 
    openstack compute service list --service nova-compute

Once that has happened, you can scan and add it to the cell using the nova-manage cell_v2 discover_hosts command:
####
    nova-manage cell_v2 discover_hosts

</details>

<details>
<summary> Error: Exceeded maximum number of retries. Exhausted all hosts available for retrying build failures for instance </summary> 

#### Solution:
Restart Some OpenStack Services
####
     systemctl restart openstack* neutron* libvirtd
Verifying Status 
#### 
     systemctl status openstack* neutron* libvirtd
Restart Neutron Services
#### 
     systemctl restart neutron*
Restart Nova-Computer Service
#### 
    service openstack-nova-compute restart
Restart Nova-Computer Service, Another Approach
#### 
    systemctl restart openstack-nova-compute.service
</details>

<details>
<summary> Error: When Adding compute Node </summary> 

 #### Error Details:
 Job for neutron-ovs-cleanup.service failed because a fatal signal was delivered causing the control process to dump core.

#### Solution:
Go to Controler Node
####
    . keysourcerc_admin
Cleanup LinuxBridge
####
    neutron-linuxbridge-cleanup

</details>

## Thank You 
