---
title: 部署Greenplum
tags: greenplum
---

## Setup CentOS7

1. version: 7.4 (*CentOS-7-x86_64-DVD-1708.iso*)
2. Software Selection when install and after install
+ when installation

```text
Minimal Install
Add-Ons: Debug tools, Compatibility Libraries, Development tools, Security Tools
```

+ after installation, modify the network configuration to connect the network。

```bash
vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
```

change `ONBOOT=no` to `ONBOOT=yes`, then `reboot`.
after restarting, install the package using `yum`

```bash
yum install net-tools ntp
```

+ language selection

```bash
ENGLISH
```

## Config the network and add static ip address

1. in the virtualbox vm, add two netword cards. One is `NAT`, the other is `Host-Only`
2. create the ifcfg-enp0s8 file

```bash
cd /etc/sysconfig/network-scripts/
cp ifcfg-enp0s3 ifcfg-enp0s8
```

3. config the static ip address. `vi ifcfg-enp0s8`, and modify the `BOOTPROTO`, `NAME`, `DEVICE`, `DEFROUTE`

```bash
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp => static
DEFROUTE=yes => no
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp0s3 => enp0s8
UUID=5040161c-a2e4-4349-a792-e27b4e2d78e1
DEVICE=enp0s3 => enp0s8
ONBOOT=yes
```

4. add the following lines

```bash
IPADDR=192.168.56.101 # ip地址
NETMASK=255.255.255.0 # 子网掩码
NETWORK=192.168.56.0 # ip段
GATEWAY=192.168.56.1 # 网关地址
BROADCAST=192.168.56.255 # 广播地址
```

5. modify the hostname

```bash
hostname mdw
echo mdw > /etc/hostname
```

6. make modification work, reboot the machine

```bash
reboot
```

## Setup Greenplum

### System Requirements

0. **File Systems**: `xfs` required for `data storage` on SUSE Linux and Red Hat (`ext3` supported for `root` file system)

1. **Disable SELinux**
+ check the status of SELinux

```bash
[root@mdw ~]# sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      28
```

+ disable the SELinux.

```bash
vi  /etc/selinux/config
```

change `SELINUX=enforcing` to `SELINUX=disabled`, then `reboot` and check the status

```bash
[root@mdw ~]# sestatus
SELinux status:                 disabled
```

2. **Disable iptables(for SUSE Linux)**

+ checks the status of iptables, if the output like following is off

```bash
# /sbin/chkconfig --list iptables
iptables 0:off 1:off 2:off 3:off 4:off 5:off 6:off
```

+ disabled the iptables, then `reboot`

```bash
# /sbin/chkconfig iptables off
```

3. **Disable firewalld(for CentOS)**

+ check the status of firewalld

```bash
[root@mdw ~]# systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2018-03-22 02:37:29 EDT; 4min 24s ago
     Docs: man:firewalld(1)
 Main PID: 762 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─762 /usr/bin/python -Es /usr/sbin/firewalld --nofork --nopid

Mar 22 02:37:28 mdw systemd[1]: Starting firewalld - dynamic firewall daemon...
Mar 22 02:37:29 mdw systemd[1]: Started firewalld - dynamic firewall daemon.
Mar 22 02:37:30 mdw firewalld[762]: WARNING: ICMP type 'beyond-scope' is not supported by the kernel for ipv6.
Mar 22 02:37:30 mdw firewalld[762]: WARNING: beyond-scope: INVALID_ICMPTYPE: No supported ICMP type., ignoring for run-time.
Mar 22 02:37:30 mdw firewalld[762]: WARNING: ICMP type 'failed-policy' is not supported by the kernel for ipv6.
Mar 22 02:37:30 mdw firewalld[762]: WARNING: failed-policy: INVALID_ICMPTYPE: No supported ICMP type., ignoring for run-time.
Mar 22 02:37:30 mdw firewalld[762]: WARNING: ICMP type 'reject-route' is not supported by the kernel for ipv6.
Mar 22 02:37:30 mdw firewalld[762]: WARNING: reject-route: INVALID_ICMPTYPE: No supported ICMP type., ignoring for run-time.
```

+ disabled the firewalld

```bash
[root@mdw ~]# systemctl stop firewalld.service
[root@mdw ~]# systemctl disable firewalld.service
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
```

+ then `reboot` and check the status

```bash
[root@mdw ~]# systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:firewalld(1)
```

### Setting the Greenplum Recommended OS Parameters

1. edit the `/etc/hosts` file and add all hostname and interface address

```bash
127.0.0.1   mdw, mdw.localdomain
192.168.56.101   mdw
192.168.56.111   sdw1
192.168.56.112   sdw2
```

2. edit the `/etc/sysctl.conf`, add following, then `reboot`

```bash
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

3. Set the following parameters in the `/etc/security/limits.conf` file

```bash
* soft nofile 65536
* hard nofile 65536
* soft nproc 131072
* hard nproc 131072
```

4. edit the `/etc/fstab` file and change the mount to `(rw,nodev,noatime,nobarrier,inode64)`

```bash
/dev/mapper/centos-root /     xfs     defaults        0 0
=>
/dev/mapper/centos-root /     xfs     nodev,noatime,nobarrier,inode64        0 0
```

5. Set blockdev (read-ahead) on a device

+ verify the read-ahead value of a disk device

```bash
[root@mdw ~]# blockdev --getra /dev/sda
8192
```

+ set blockdev (read-ahead) on a device

```bash
[root@mdw ~]# blockdev --setra 16384 /dev/sda
[root@mdw ~]# blockdev --getra /dev/sda
16384
```

+ centos7: if blockdev size reset back to 8192 after system reboot, please set:

```bash
# echo '/sbin/blockdev --setra 16384 /dev/sda' >> /etc/rc.d/rc.local
# chmod u+x /etc/rc.d/rc.local
```

6. set linux disk I/O scheduler for disk access, and the `deadline` scheduler option is recommended

```bash
[root@mdw ~]# echo deadline > /sys/block/sda/queue/scheduler
[root@mdw ~]# cat /sys/block/sda/queue/scheduler
noop [deadline] cfq
```

7. specify the I/O scheduler at boot time on systems that use grub2

```bash
[root@mdw ~]# grubby --update-kernel=ALL --args="elevator=deadline"
[root@mdw ~]# grubby --info=ALL
index=0
kernel=/boot/vmlinuz-3.10.0-693.el7.x86_64
args="ro crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet LANG=en_US.UTF-8 elevator=deadline"
root=/dev/mapper/centos-root
initrd=/boot/initramfs-3.10.0-693.el7.x86_64.img
title=CentOS Linux (3.10.0-693.el7.x86_64) 7 (Core)
index=1
kernel=/boot/vmlinuz-0-rescue-a4ae0d7e26fb45f5b5208bed38b11f20
args="ro crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet elevator=deadline"
root=/dev/mapper/centos-root
initrd=/boot/initramfs-0-rescue-a4ae0d7e26fb45f5b5208bed38b11f20.img
title=CentOS Linux (0-rescue-a4ae0d7e26fb45f5b5208bed38b11f20) 7 (Core)
index=2
non linux entry
```

8. Disable Transparent Huge Pages (THP)

```bash
[root@mdw ~]# grubby --update-kernel=ALL --args="transparent_hugepage=never"
[root@mdw ~]# cat /sys/kernel/mm/*transparent_hugepage/enabled
always madvise [never]
```

9. Disable IPC object removal for RHEL 7.2 or CentOS 7.2. Edit `/etc/systemd/logind.conf` and umcomment or add `RemoveIPC=no`, then restart the service to make it work

```bash
service systemd-logind restart
```

### Creating the Greenplum Database Administrative User Account(Master Only)

```bash
# groupadd gpadmin
# useradd gpadmin -g gpadmin
# passwd gpadmin
New password: <changeme>
Retype new password: <changeme>
```

### Installing the Greenplum Database Software(Master Only)

1. Log in as `root` on the machine that will become the Greenplum Database master host
2. Upload the Binary Distribution
3. Launch the installer using bash

```bash
[root@mdw ~]# bash greenplum-db-5.5.0-rhel7-x86_64.bin
```

4. If you installed as root, change the ownership and group of the installed files to gpadmin

```bash
[root@mdw ~]# chown -R gpadmin /usr/local/greenplum*
[root@mdw ~]# chgrp -R gpadmin /usr/local/greenplum*
```

### Installing and Configuring Greenplum on all Hosts

1. **To install and configure Greenplum Database on all specified hosts**
+ Log in to the master host as gpadmin

```bash
[root@mdw ~]# su -
```

+ Source the path file from your master host's Greenplum Database installation directory

```bash
[root@mdw ~]# source /usr/local/greenplum-db/greenplum_path.sh
```

+ Create a file called hostlist with all hosts as following and seglist without mdw in `/home/gpadmin/gpconfig/` dir.

```text
mdw
sdw1
sdw2
```

+ Config ssh key exchange(master server and gpadmin user)

```bash
[root@mdw ~]# gpssh-exkeys -f /home/gpadmin/gpconfig/hostlist
```

+ Run the `gpseginstall` utility referencing the hostfile_exkeys file you just created

```bash
[root@mdw ~]# gpseginstall -f /home/gpadmin/gpconfig/seglist
```

+ Confirm installation

```bash
[root@mdw ~]# su - gpadmin
[gpadmin@mdw ~]$ gpssh -f ~/gpconfig/hostlist -e ls -l $GPHOME
```

2. **Creating the Data Storage Areas**

+ To create the data directory location on the master

```bash
[root@mdw /]# mkdir -p /data/master
[root@mdw /]# chown gpadmin /data/master
```

+ Creating Data Storage Areas on Segment Hosts

```bash
[root@mdw /]# source /usr/local/greenplum-db/greenplum_path.sh
[root@mdw /]# gpssh -f /home/gpadmin/gpconfig/seglist -e 'mkdir -p /data/primary'
[sdw2] mkdir -p /data/primary
[sdw1] mkdir -p /data/primary
[root@mdw /]# gpssh -f /home/gpadmin/gpconfig/seglist -e 'mkdir -p /data/mirror'
[sdw2] mkdir -p /data/mirror
[sdw1] mkdir -p /data/mirror
[root@mdw /]# gpssh -f /home/gpadmin/gpconfig/seglist -e 'chown gpadmin /data/primary'
[sdw2] chown gpadmin /data/primary
[sdw1] chown gpadmin /data/primary
[root@mdw /]# gpssh -f /home/gpadmin/gpconfig/seglist -e 'chown gpadmin /data/mirror'
[sdw2] chown gpadmin /data/mirror
[sdw1] chown gpadmin /data/mirror
```

3. **Synchronizing System Clocks**

+ on master, edit `/etc/ntp.conf` file, set server to the data center's NTP time server

```bash
server 202.108.6.95       #cn.ntp.org.cn
```

+ On each segment host, edit the `/etc/ntp.conf` file and set as following

```bash
server mdw prefer
server smdw
```

+ On the standby master host, edit the /etc/ntp.conf file and set as following

```bash
server mdw prefer
server 202.108.6.95
```

+ On the master host, use the NTP daemon synchronize the system clocks on all Greenplum hosts.

```bash
[root@mdw /]# gpssh -f /home/gpadmin/gpconfig/hostlist -v -e 'ntpd'
[WARN] Reference default values as $MASTER_DATA_DIRECTORY/gpssh.conf could not be found
Using delaybeforesend 0.05 and prompt_validation_timeout 1.0

[Reset ...]
[INFO] login mdw
[INFO] login sdw2
[INFO] login sdw1
[ mdw] ntpd
[sdw2] ntpd
[sdw1] ntpd
[INFO] completed successfully

[Cleanup...]
```

+ enable ntpd service autostart( all host)

```bash
[root@mdw /]# gpssh -f /home/gpadmin/gpconfig/hostlist -e 'systemctl disable chronyd.service'
[root@mdw /]# gpssh -f /home/gpadmin/gpconfig/hostlist -e 'systemctl enable ntpd.service'
[root@mdw /]# gpssh -f /home/gpadmin/gpconfig/hostlist -e 'systemctl start ntpd.service'
```

4. Validating Your Systems

+ Validating OS Settings

```bash
gpcheck -f ~/gpconfig/hostlist -m mdw
```

5. Initializing a Greenplum Database System

+ Log in as gpadmin

```bash
[root@mdw /]# su - gpadmin
Last login: Thu Mar 22 05:09:20 EDT 2018 on pts/0
[gpadmin@mdw ~]$
```

+ Make a copy of the gpinitsystem_config file to use as a starting point

```bash
[gpadmin@mdw ~]$ cp /usr/local/greenplum-db/docs/cli_help/gpconfigs/gpinitsystem_config ~/gpconfig/gpinitsystem_config
```

+ run the initialization utility

```bash
[gpadmin@mdw ~]$ gpinitsystem -c gpconfig/gpinitsystem_config -h gpconfig/seglist
```

+ see the `Greenplum Database instance successfully created.` You install successfully