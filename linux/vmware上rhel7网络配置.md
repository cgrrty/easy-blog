# vmware上rhel7网络配置
> 打算试试rhel7，基本的需求就是平时装在我的windows电脑上，要让macbook能够ssh连接上，这样办公，学习都比较方便。vmware快速安装rhel7又快又轻松，就配置一个用户密码，很快就装好了，比原来装redhat9，suse11什么的方便多了。  

* 一上来就遇到一个问题，虚拟机的设置中已经显示网卡已连接了，然并卵，没有看到eth0网卡。仔细在ifconfig里看了下，有一块叫做eno16777736的网卡。为什么网卡名称不对呢？  
在[rhel7网络模块的文档](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Networking_Guide/ch-Consistent_Network_Device_Naming.html)中，我们可以看到原因。  
在rhel7中，网卡的命名方式被改变了，究其原因，想来是因为eth[0123...]的命名方式被用户吐槽太多了，这种命名方式实际用户场景中网卡的label无关，无法帮助用户来准确地确定网卡，如果涉及到网卡切换和高端服务器（32路机必然很多网卡吧。。）多网卡的场景，有点麻烦。所以，redhat的工程师兄弟们在rhel7中设计了一套新的命名方案（ The default is to assign fixed names based on firmware, topology, and location information. This has the advantage that the names are fully automatic, fully predictable, that they stay fixed even if hardware is added or removed (no re-enumeration takes place), and that broken hardware can be replaced seamlessly.）  
命名方案
    * 1.使用固件或者BIOS中为板载设备提供的序列号(例如:eno1)，如果不行，使用方案2
    * 2.使用固件或者BIOS中为PICE热拔插槽位提供的序列号（例如：ens1），如果不行，使用方案3
    * 3.使用硬件连接处的槽位号（例如：enp2s0），如果不行，使用方案5
    * 4.使用设备的mac地址（例如：enx78e7d1ea46da），该方案仅当用户主动选择时才被使用，默认不采用
    * 5.如果1，2，3全部不行的话，使用传统的不可预测的内核命名方案（例如：eth0）
    如果用户使用biosdevname=1的设置开启了，biosdevname将被使用。如果用户配置了udev规则，那么优先使用udev规则。  
* 这里，虚拟机通过桥接模拟出来的虚拟网卡是可以得到板载设备的序列号的，en代表ethernet，o代表onboard，它的序列号是16777736。然而为什么时这个数字呢？  
如果这个虚拟网卡的信息时vmware写入的，那么根据规则，这个信息有可能写在bios里，进到虚拟机的bios中查看了一番，没有什么有用的信息。
当然会有一些有探究精神的人探究过这个问题，这里有[一篇文章](http://serverfix.net/why-is-my-eth0-called-eno16777736/)讲的非常详细。
通过检查/sys/class/net/eno16777736我们可以看到它是一个软链接  
eno16777736 -> ../../devices/pci0000:00/0000:00:11.0/0000:02:01.0/net/eno16777736  
在这个文件夹中的acpi_index中存储着16777736，它是该虚拟设备的acpi_index。这个acpi_index是固件给予pci设备的实例序列号。  
<code>
What:       /sys/bus/pci/devices/.../acpi_index  
Date:       July 2010  
Contact:    Narendra K <narendra_k@dell.com>, linux-bugs@dell.com  
Description:  
        Reading this attribute will provide the firmware
 given instance (ACPI _DSM instance number) of the PCI device.  
        The attribute will be created only if the firmware has given an instance number to the PCI device. ACPI _DSM instance number will be given priority if the system firmware provides SMBIOS type 41 device type instance also.
</code>  
甚至我们可以查到该acpi_index对应的pci槽位号。
* 好吧，我还是想按自己的习惯来，把eno16777736改成eth0吧。
  具体方法就是按照之前的方案，关闭biosdevname，使用udev规则即可。
  [参见](http://blog.sina.com.cn/s/blog_926acdf80102vsc1.html)
* 这样，网卡名称就修改完了。考虑到要与局域网内的另外一台主机进行通信，所以虚拟机中的网络模式采用的是桥接，那么下一步就是给rhel7设置一个固定ip。
* 在/etc/sysconfig/network-scripts/ifcfg-eth0中修改IPADDR，NETMASK,GATEWAY，并把BOOTPROTO设为static，将ONBOOT设为yes。service network restart，重启network服务，ping 233.5.5.5通了。
* 这时候可以再设置一下dns，修改/etc/resolv.conf文件，将nameserver设为233.5.5.5（阿里dns）。ping www.baidu.com 。ok
* 到这里如果从mac上能ping通虚拟机，但是ssh无法连上的话，考虑关闭防火墙服务，service iptables stop. 
* ok，成功为vmware中的rhel7做好a网络配置了。