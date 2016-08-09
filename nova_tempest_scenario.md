Nova Tempest Scenario Test
======
Tempest is the essential test framework for OpenStack test, it contains variable tests such as 
scenario test, api test...Scenario test simulates user using OpenStack normally. Here we will 
do some essential tests for Nova.
Where to see the log info(LOG.debug in tempest code)?
In tempest.log, it will output the log information with REST and json information.
#Test_minimum_basic
Use image cirros-d160722-aarch64-disk.img, from: 
http://download.cirros-cloud.net/daily/20160722
If you want to upload it to glance, use command below.This is not essential
```shell
glance image-create --name cirros --disk-format qcow2 --container-format bare --visibility public --file cirros-d160722-aarch64-disk.img --progress
glance image-update --property hw_firmware_type=uefi --property short_id=ubuntu16.04 e3905bd8-a127-4cb3-9661-58d3bf65b7db
```
##Configure file
Modify the tempest/etc/tempest.conf:
```shell
flavor_ref = 1
[scenario]
img_file = cirros-d160722-aarch64-disk.img
aki_img_file =
ari_img_file =
ami_img_file =
img_dir = /opt/stack
img_properties = hw_firmware_type:uefi, short_id:ubuntu16.04
```
##Run testr
Run testr in tempest directory:
```shell
testr init
testr run tempest.scenario.test_minimum_basic
```
##issue
1.Boot image got problem, console log output info as below:
Synchronizaiton failed 0x000000021324
That is due to the flavor, default is flavor_ref=42 --> m1.nano. But nano will not suitable for 
AArch64 cirros. Should modify it to 1(m1.tiny)
2.Check partitions error:
check_partitions will call "cat /proc/partitions", and calculate times of the device name occurrence.
The cirros image for AArch64,/proc/partitions:
```shell
   1        0      65536 ram0
   1        1      65536 ram1
   1        2      65536 ram2
   1        3      65536 ram3
   1        4      65536 ram4
   1        5      65536 ram5
   1        6      65536 ram6
   1        7      65536 ram7
   1        8      65536 ram8
   1        9      65536 ram9
   1       10      65536 ram10
   1       11      65536 ram11
   1       12      65536 ram12
   1       13      65536 ram13
   1       14      65536 ram14
   1       15      65536 ram15
```
But in x86_64 cirros:
```shell
253        0   20971520 vda
11        0        422 sr0
```
#Other passed scenario test:
* test_server_basic_ops
* test_server_advanced_ops
* test_aggregates_basic_ops
* test_encrypted_cinder_volumes
* test_security_groups_basic_ops
* test_shelve_instance
* test_stamp_pattern
* test_volume_boot_pattern

#test_volume_boot_pattern
Make sure that the flavor_ref = 1 in tempest.conf
If the flavor disk > 1G, will cause error:
```shell
Details: {u'message': u'Volume is smaller than the minimum size specified in image metadata. Volume size is 1073741824 bytes, minimum size is 5368709120 bytes.', u'code': 400}
```
So make sure that the disk image size < 1G will OK
