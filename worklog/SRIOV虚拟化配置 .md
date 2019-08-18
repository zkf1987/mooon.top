## 1、在BIOS中开启硬件虚拟化支持
## 2、开启IOMMU支持（判断是否开启dmesg | grep -e DMAR -e IOMMU）
    a) 在/etc/default/grub文件,添加intel_iommu=on参数,

    b) 运行命命令grub2-mkconfig -o /boot/grub2/grub.cfg

    c) 重启计算机
## 3、启动SRIVO内核模块   modprobe igb（查看 ethtool -i 网卡）
## 4、激活虚拟功能VF    modprobe igb max_vfs=7
    千兆网卡最多支持8个vf0-7,千兆网卡目前支持比较好的是INTEL I350， 82576S虽然也支持SRIOV但是只支持虚拟机是        linux的情况，windows系统不支持；
    万兆网卡最多支持64个vg0-63，intel的新新一代万兆网卡都支持SRIOV x520 x540等；
    如果需要重新设置vf 可以删除模块在重新加载
    modprobe -r igb
## 5、将配置永久写入配置文件
```shell
    echo "options igb max_vfs=7" >>/etc/modprobe.d/igb.conf
```
## 6、修改/sys/bus/pci/devices/0000\:02\:00.1/sriov_numvfs 的数量  最多不能超过上边设置的max_vfs的数量，设置虚拟网卡的数量
    0000\:02\:00.1为lspci或ethtool -i 网卡  时看到的pci号
## 7、lspci | grep 网卡型号
    出现 Ethernet Controller Virtual Function 即为开启成功
## 8、修改虚拟机配置文件增加一下内容
```shell
<interface type='hostdev' managed='yes'>
<source>
<address type='pci' domain='0x0' bus='0x00' slot='0x07' function='0x0'/>
</source>
<mac address='52:54:00:6d:90:02'/>
</interface>
```