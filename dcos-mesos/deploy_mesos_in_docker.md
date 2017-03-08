Deploy Mesos cluster in Docker
============
This article will tell you how to deploy Mesos in Docker to taste its functions. Deploy Mesos needs some VMs originally, in this article we can do all this work in only one VM with some docker container. The docker images is pushed to the registory.
#Prepare
Need 4 docker images at least:
* Zooper
* Mesos Master
* Marathon
* Mesos Slave 
Host: Ubuntu15.04 Vivid, X86_64(VM)

#Get host IP
    Get host VM ip, and export to env:
```shell
$ HOST_IP=192.168.100.146
```
#Launch the ZooKeeper Container
```shell
docker run -d \
-p 2181:2181 \
-p 2888:2888 \
-p 3888:3888 \
garland/zookeeper
```
You don't need to set the configure file fot zooKeeper. The image has done it for you. The same as Mesos master and slave ,Marathon image.

#Launch Mesos Master Container
```shell
docker run --net="host" \
-p 5050:5050 \
-e "MESOS_HOSTNAME=${HOST_IP}" \
-e "MESOS_IP=${HOST_IP}" \
-e "MESOS_ZK=zk://${HOST_IP}:2181/mesos" \
-e "MESOS_PORT=5050" \
-e "MESOS_LOG_DIR=/var/log/mesos" \
-e "MESOS_QUORUM=1" \
-e "MESOS_REGISTRY=in_memory" \
-e "MESOS_WORK_DIR=/var/lib/mesos" \
-d \
garland/mesosphere-docker-mesos-master
```
Also, in the officially installation procedures, we should set the Mesos zk and mesos quorum. But it easily for docker to do it its self. All we should do is pass the parameters to it.

#Launch Marathon
docker run -d \
-p 8080:8080 \
garland/mesosphere-docker-marathon --master zk://${HOST_IP}:2181/mesos \
--zk zk://${HOST_IP}:2181/marathon

#Launch Mesos Slave
```shell
docker run -d \
--name mesos_slave_1 \
--entrypoint="mesos-slave" \
-e "MESOS_MASTER=zk://${HOST_IP}:2181/mesos" \
-e "MESOS_LOG_DIR=/var/log/mesos" \
-e "MESOS_LOGGING_LEVEL=INFO" \
garland/mesosphere-docker-mesos-master:latest
```
The Mesos Slave and the master share the same image. We can --entrypoint to let it run as a slave.

#Access to Mesos
```shell
http://${HOST_IP}:5050
```

#Start a job through Marathon
The website is http://${HOST_IP}:5050.
Marathon can deploy a long term application job to the Slave container. 
Click right top button in the Marathon. Create a job/task, the command is 
```shell
echo "hello" >>/tmp/output.txt
```
The we can check whether the task is running by enter into the container. Then you can see "hello" will be adding to this file every second.
