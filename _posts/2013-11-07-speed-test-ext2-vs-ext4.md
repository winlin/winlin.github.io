---
layout: post
title: "Speed Test Ext2 VS Ext4"
description: "Speed Test Ext2 VS Ext4"
category: Testing
tags: 
---
{% include JB/setup %}

最近需要定制一款Android专用设备，对于分区文件系统面临两个选择：Ext2 or Ext4.  
对于Ext4有哪些功能特性参见 [《Linux Kernel Ext4 Document》](https://www.kernel.org/doc/Documentation/filesystems/ext4.txt), 为方便查阅，如下：  

* ability to use filesystems > 16TB (e2fsprogs support not available yet)
* extent format reduces metadata overhead (RAM, IO for access, transactions)
* extent format more robust in face of on-disk corruption due to magics,
* internal redundancy in tree
* improved file allocation (multi-block alloc)
* lift 32000 subdirectory limit imposed by i_links_count
* nsec timestamps for mtime, atime, ctime, create time
* inode version field on disk (NFSv4, Lustre)
* reduced e2fsck time via uninit_bg feature
* journal checksumming for robustness, performance
* persistent file preallocation (e.g for streaming media, databases)
* ability to pack bitmaps and inode tables into larger virtual groups via the
  flex_bg feature
* large file support
* Inode allocation using large virtual block groups via flex_bg
* delayed allocation
* large block (up to pagesize) support
* efficient new ordered mode in JBD2 and ext4(avoid using buffer head to force
  the ordering)


首先，有一点是十分明确地：Ext2由于没有journal功能，所以，其读写速度应当高于Ext4, 但是也正是因为此，当系统突然断电，进行fsck时，会对整个文件系统进行检查，会占用很长的时间；此外，数据的完整性也缺乏保障。 虽然，现在的Android手机分区都会使用Ext4文件系统，但是，为了追求最高的性能，还得对Ext2进行对比以后才能做出最后的决定。  
对于二者的比较，网上也有其他比较，有的比较结果显示Ext4不论是否在开启journal功能时，读写速度都会比Eext要好，其中使用IOZone benchmark工具进行的[比较](http://en.community.dell.com/techcenter/high-performance-computing/w/wiki/2290.aspx)如下一项比较如下：  
 ![](/images/IOZone_read_prefoemance.png)   
 ![](/images/IOZone_write_prefoemance.png)   
 
测试的方法是，分别创建出一个1GB的Ext2、Ext4磁盘，然后使用不同的 mount 选项来分别挂载这两个磁盘，重复多次向两个磁盘中多次写入长度不同的数据，每次写入的数据从 10B--1000000B 不等，完全随机；没写完一个文件以后，再按照行来读取整个文件；分别累计每次读写的时间，最后根据读写总量和时间来算出平均读写速度。

下面是概要测试结果：  
测试环境： Ubuntu 12.04 x64
{% highlight c++ %}
1. 直接按照默认的方式挂载 ext2 格式的磁盘 mount ext2 without any flag
read   31448390 Bytes(30711.000000 KB) using  33.000000 seconds SPEED=905.000000 KB/s   
write   31448390 Bytes(30711.000000 KB) using  39.000000 seconds SPEED=779.000000 KB/s

2. 直接按照默认方式挂载 ext4 格式的磁盘 mount ext4 without any flag
read   26013420 Bytes(25403.000000 KB) using  30.000000 seconds SPEED=819.000000 KB/s  
write   26013420 Bytes(25403.000000 KB) using  35.000000 seconds SPEED=708.000000 KB/s

3. 有优化选项的挂载 ext4 格式磁盘
3.1 mount ext4 with flag:(rw,noatime,errors=remount-ro,commit=8)
read   29472330 Bytes(28781.000000 KB) using  32.000000 seconds SPEED=874.000000 KB/s  
write   29472330 Bytes(28781.000000 KB) using  38.000000 seconds SPEED=751.000000 KB/s

3.2 mount ext4 with flag:(rw,noatime,errors=remount-ro,commit=5)
read   29507100 Bytes(28815.000000 KB) using  33.000000 seconds SPEED=864.000000 KB/s  
write   29507100 Bytes(28815.000000 KB) using  38.000000 seconds SPEED=750.000000 KB/s

3.3 mount ext4 with flag:(rw,noatime,errors=remount-ro,data=writeback,commit=5)
read   34157500 Bytes(33356.000000 KB) using  34.000000 seconds SPEED=954.000000 KB/s  
write   34157500 Bytes(33356.000000 KB) using  40.000000 seconds SPEED=813.000000 KB/s

4. 关闭日志功能且有优化选项的挂载 ext4 格式磁盘
4.1 mount ext4 without journal with flag:(rw,noatime,barrier=0,data=writeback,errors=remount-ro,commit=5)
read   36812960 Bytes(35950.000000 KB) using  37.000000 seconds SPEED=965.000000 KB/s  
write   36812960 Bytes(35950.000000 KB) using  43.000000 seconds SPEED=821.000000 KB/s

4.2 mount ext4 without journal with flag:(rw,noatime,barrier=0,data=order,errors=remount-ro,commit=5)
read   27964560 Bytes(27309.000000 KB) using  31.000000 seconds SPEED=862.000000 KB/s  
write   27964560 Bytes(27309.000000 KB) using  36.000000 seconds SPEED=744.000000 KB/s
{% endhighlight %}
#### 根据测试结果得出的结论是：
* 默认状态下 ext4 读速度 要比 ext2 慢 100KB/S 左右； 写速度 要比 ext2 慢 70KB/S 左右
* 有挂载优化选项的 ext4  读速度 要比 ext2 慢 50KB/S 左右； 写速度 要比 ext2 慢 30KB/S 左右，其中data=writeback可以明显提升读写速度
* 有挂载优化选项且关闭日志功能的 ext4  读速度 要比 ext2 快 50KB/S 左右； 写速度 要比 ext2 快 40KB/S 左右，其中data=writeback可以明显提升读写速度

综上，并参照其他网站提供测试结果，ext2和ext4 在默认挂载的情况下差别仍可接受，为保证数据可靠性和最优性能，
建议采用 “有优化选项的挂载 ext4 格式磁盘的方式”，即 3.2 测试用例，挂载选项应包括：rw,noatime,errors=remount-ro,commit=5  

  commit 等选项的含义参看[《Linux Kernel Ext4 Document》](https://www.kernel.org/doc/Documentation/filesystems/ext4.txt)
    
如果你想获得详细的测试脚本，请移步：[Ext2-Ext4SpeedTest](https://github.com/winlin/Ext2-Ext4SpeedTest)    


