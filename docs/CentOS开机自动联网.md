# CentOS开机自动联网

通过修改配置文件将ONBOOT='no'修改为yes即可。

步骤如下：
1. 查看当前的网络接口名称
2. 进入`network-scripts`目录找到连接有网线的网络端口的名称比如：ifcfg-eth0，将其的`ONBOOT="No"`修改为`"yes"`。
3. 重启机器即可完成

命令如下：
```
# ifconfig -a
# cd /etc/sysconfig/network-scripts/
# vi ifcfg-eth0
# reboot
```