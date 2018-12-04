# 02_GP部署Lite

## Install and Configurate OS
1. Version：CentOS7.4 (*CentOS-7-x86_64-DVD-1708.iso*)
2. Software Selection：Minimal Install，Add-Ons: Debug tools, Compatibility Libraries, Development tools, Security Tools
3. Configurate the Network
   + NAT + Host-Only
   + copy ifcfg-enp0s3 to ifcfg-enp0s8 which is Host-Only card configuration
   + Host-Only network card is static
   + NAT network card is dhcp
   + modify: `BOOTPROTO`, `NAME`, `DEVICE`, `DEFROUTE`, `ONBOOT`, if static, add ip configuration
4. install the net-tools and ntp
```
# yum install net-tools ntp
```
5. modify the hostname
```
# echo mdw > /etc/hostname
```
6. Disable SELinux: edit `/etc/selinux/config`, modify `SELINUX=enforcing` to `SELINUX=disabled`
7. Disable firewalld
```
# systemctl stop firewalld.service
# systemctl disable firewalld.service
```
8. Edit the `/etc/hosts` file and add all hostname and interface address
9. Edit the `/etc/sysctl.conf`, add following lines
```
kernel.shmmax = 500000000
kernel.shmmni = 4096
kernel.shmall = 4000000000
kernel.sem = 250 512000 100 2048
kernel.sysrq = 1
kernel.core_uses_pid = 1
kernel.msgmnb = 65536
kernel.msgmax = 65536
kernel.msgmni = 2048
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_max_syn_backlog = 4096
net.ipv4.conf.all.arp_filter = 1
net.ipv4.ip_local_port_range = 10000 65535
net.core.netdev_max_backlog = 10000
net.core.rmem_max = 2097152
net.core.wmem_max = 2097152
vm.overcommit_memory = 2
```
10. Edit `/etc/security/limits.conf` file and add following
```
* soft nofile 65536
* hard nofile 65536
* soft nproc 131072
* hard nproc 131072
```
11. Edit `/etc/fstab` file and change `defaults` to `nodev,noatime,nobarrier,inode64`
12. Set blockdev (read-ahead) on a device
```
# echo '/sbin/blockdev --setra 16384 /dev/sda' >> /etc/rc.d/rc.local
# chmod u+x /etc/rc.d/rc.local
```
13. Specify the I/O scheduler at boot time on systems that use grub2
```
# grubby --update-kernel=ALL --args="elevator=deadline"
```
14. Disable Transparent Huge Pages (THP)
```
# grubby --update-kernel=ALL --args="transparent_hugepage=never"
```
15. Edit `/etc/systemd/logind.conf` and umcomment or add `RemoveIPC=no`
16. Configurate ntp.
    + On master, edit `/etc/ntp.conf` file, add `server 202.108.6.95`
    + On each segment host, edit the `/etc/ntp.conf` file and add `server mdw prefer`
    + On standby host, edit the `/etc/ntp.conf` file and add `server mdw prefer`
    + Enable ntpd service autostart
```
# systemctl disable chronyd.service
# systemctl enable ntpd.service
# systemctl start ntpd.service
```
17. reboot the os and make change work
```
# reboot
```

## Install Greenplum Database
1. Create the Greenplum Database Administrative User Account(Master Only)
```
# groupadd gpadmin
# useradd gpadmin -g gpadmin
# passwd gpadmin
New password: <changeme>
Retype new password: <changeme>
```
2. Install the Greenplum Database Software(Master Only)
   + Upload the Binary Distribution
   + exec the bin file. `# bash greenplum-db-5.5.0-rhel7-x86_64.bin`
   + change the ownership and group of the installed files to gpadmin
```
# chown -R gpadmin /usr/local/greenplum*
# chgrp -R gpadmin /usr/local/greenplum*
```
3. Install and configure Greenplum Database on all specified hosts
   - Source the path file to support greenplum env. `# source /usr/local/greenplum-db/greenplum_path.sh`
   - Create gpconfig dir to store the gp config. `# mkdir /home/gpadmin/gpconfig`
   - Create `hostlist` file which add all hosts and `seglist` file which add all segment nodes in the gpconfig dir.
   - Configurate ssh key exchange. `# gpssh-exkeys -f /home/gpadmin/gpconfig/hostlist`
   - Run the `gpseginstall` utility. `# gpseginstall -f /home/gpadmin/gpconfig/seglist`
   - Change the ownership and group of gpconfig to gpadmin
```
# chown -R gpadmin /home/gpadmin/gpconfig
# chgrp -R gpadmin /home/gpadmin/gpconfig
```
4. Creating the Data Storage Areas
+ To create the data directory location on the master
```
# mkdir -p /data/master
# chown gpadmin /data/master
```
+ Creating Data Storage Areas on Segment Hosts
```
# source /usr/local/greenplum-db/greenplum_path.sh
# gpssh -f /home/gpadmin/gpconfig/seglist -e 'mkdir -p /data/primary'
# gpssh -f /home/gpadmin/gpconfig/seglist -e 'mkdir -p /data/mirror'
# gpssh -f /home/gpadmin/gpconfig/seglist -e 'chown gpadmin /data/primary'
# gpssh -f /home/gpadmin/gpconfig/seglist -e 'chown gpadmin /data/mirror'
```
5. Synchronizing System Clocks
```
# gpssh -f /home/gpadmin/gpconfig/hostlist -v -e 'ntpd'
```
6. add greenplum path in the gpadmin bashrc
```
echo 'source /usr/local/greenplum-db/greenplum_path.sh' >> /home/gpadmin/.bashrc
```
7. Initializing a Greenplum Database System
+ Log in as gpadmin
```
# su - gpadmin
```
+ Make a copy of the gpinitsystem_config file to use as a starting point
```
$ cp /usr/local/greenplum-db/docs/cli_help/gpconfigs/gpinitsystem_config ~/gpconfig/gpinitsystem_config
```
+ modify the port as following
```
PORT_BASE = 6000
MIRROR_PORT_BASE = 7000
REPLICATION_PORT_BASE = 8000
MIRROR_REPLICATION_PORT_BASE = 9000
```
+ run the initialization utility
```
$ gpinitsystem -c gpconfig/gpinitsystem_config -h gpconfig/seglist
```
+ see the `Greenplum Database instance successfully created.` You install successfully
+ add `MASTER_DATA_DIRECTORY` env in the bashrc file
```
$ echo 'export MASTER_DATA_DIRECTORY=/gpdata/master/gpseg-1' >> ~/.bashrc
```