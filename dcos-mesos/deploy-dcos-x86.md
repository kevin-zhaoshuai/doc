Depoloyment of DC/OS on X86_64
======
#Host:
Ubuntu 15.04 vivid, all the node are in the Guest VM. All the guest use the network NAT to eno1 to HP server. All the Guest has access to internet. 192.168.100.***.Set 1 master node , 2 slave nodes and 1 node as bootstrap node. 
#First, set bootstrap node:
* $ ssh to the bootstrap node
* $ mkdir -p genconf
* Create the config.yaml. In the genconf, generate the file as below. Pay attention that the resolvers you can reference to /etc/resolv.conf. Master list you can add more master nodes and list all the master ip here. 
```shell
	---
	bootstrap_url: http://<bootstrap node ip>:<port, usually 8080>
	cluster_name: 'DC/OS'
	exhibitor_storage_backend: static
	ip_detect_filename: /genconf/ip-detect
	master_list:
	- <master node ip1>
	resolvers:
	- 192.168.100.1
	oauth_enabled: 'false'
	telemetry_enabled: 'false'
```

* Set ip-detect script, I use:

```shell
	#!/usr/bin/env bash
	set -o nounset -o errexit
	export PATH=/usr/sbin:/usr/bin:$PATH
	echo $(ip addr show eth0 | grep -Eo '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' | head -1)
```

* Get the DCOS installer, in genconf:

```shell
	$ cd ../
	$ curl -O https://downloads.dcos.io/dcos/EarlyAccess/dcos_generate_config.sh
```

* Install Docker and run , <your-port> is the port in config.yaml of "bootstrap_url"

```shell
	$ sudo apt-get install docker.io
	$ sudo docker run -d -p <your-port>:80 -v $PWD/genconf/serve:/usr/share/nginx/html:ro nginx
```
#Second, set the master Node:
```shell
ssh <master-ip>
mkdir /tmp/dcos && cd /tmp/dcos
curl -O http://<bootstrap-ip>:<your_port>/dcos_install.sh
sudo bash dcos_install.sh master
```
* May meet some problem here. Can not load the service. Use "journal -xe" to see more details. Usually ,we should install docker.io, unzip and ipset. Also, make sure /usr/bin/mkdir /usr/bin/ln /usr/bin/tar are exactly existing. Or systemd will report error. If /bin/mkdir , you should manually soft link it at /usr/bin/mkdir

#Third, set the slave node:
```shell
	1. ssh <master-ip>         
	2. mkdir /tmp/dcos && cd /tmp/dcos         
	3. curl -O http://<bootstrap-ip>:<your_port>/dcos_install.sh         
	4. sudo bash dcos_install.sh slave
```
If all done, you can see the master nodes is visible in the link:
```shell
http://<master-ip>:8181/exhibitor/v1/ui/index.html
Launch the DC/OS web interface at: http://<master-node-public-ip>/
```
#Reference:
```shell
1. Advanced DCOS installation Guide
https://dcos.io/docs/1.7/administration/installing/custom/advanced/
2. Trobleshooting:
https://dcos.io/docs/1.7/administration/installing/custom/troubleshooting/
3. http://www.cnblogs.com/jackluo/p/5431730.html
```
