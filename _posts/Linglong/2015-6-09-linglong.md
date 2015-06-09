---
layout: post
title: linglong
category: 玲珑自定义
tags: Linglong
keywords: 这他妈什么玩意
description: 
---

>**Data:2015/3/29 6:02:27**

### 适用于pxe+kickstart环境
>
-  存在问题，虚拟机(VirtualBox/VMware)测试均无问题，但在PC实验，中途进行中断，此问题正在进行调试中。。。

	#version=RHEL7
	# Firewall configuration
	firewall --disabled
	# System authorization information
	auth --enableshadow --passalgo=sha512
	# Use network installation
	url --url="http://192.168.1.106/cobbler/ks_mirror/centos7.0-x86_64"
	# disable the Setup Agent on first boot
	firstboot --enabled
	ignoredisk --only-use=sda
	# Use text mode install
	text
	# Keyboard layouts
	keyboard us
	#--vckeymap=us --xlayouts='cn'
	# System language
	lang en_US
	# SELinux configuration
	selinux --disabled
	# Installation logging level
	logging --level=info

	# Reboot after installation
	reboot

	# Network information
	network  --bootproto=dhcp --activate
	#network  --hostname=localhost.localdomain
	# Root password
	rootpw --iscrypted $6$6.ZOFbx0wK7k8Tk5$cXI9VL45yCuAXdENSQ8CTXTmi0qygN5sDhTCJ4osT9nqhhTZEatYMGWi1wgd6hI..sNcFXtVV22HRuSYmSd5.0
	# System timezone
	timezone Asia/Shanghai
	#--isUtc --nontp --ntpservers=0.centos.pool.ntp.org,1.centos.pool.ntp.org,2.centos.pool.ntp.org,3.centos.pool.ntp.org
	# System bootloader configuration
	bootloader --location=mbr --boot-drive=sda
	# Clear the Master Boot Record
	zerombr
	# Partition clearing information
	clearpart --none --initlabel 
	# Disk partitioning information
	part /boot --fstype="xfs" --ondisk=sda --size=431
	part swap --fstype="swap" --ondisk=sda --size=2048
	part / --fstype="xfs" --ondisk=sda --size=18000

	%packages
	@base
	@compat-libraries
	@core
	@development
	lrzsz
	%end

	%post --interpreter /bin/bash
	IP=`ifconfig enp0s3 | grep inet | grep -v inet6 | awk -F[' ':]+ '{print $3}'`
	MASK=`ifconfig enp0s3 | grep inet | grep -v inet6 | awk -F[' ':]+ '{print $5}'
	GATE=`route -n | grep UG | awk '{print $2}'`
	sed -i "s@\(^H.*=\).*@\1server-$IP@g" /etc/sysconfig/network
	hostnamectl --static set-hostname server-$IP
	sed -i "s@\(^BOOT.*=\).*@\1static@g" /etc/sysconfig/network-scripts/ifcfg-enp0s3
	echo "IPADDR=$IP" >> /etc/sysconfig/network-scripts/ifcfg-enp0s3
	echo "NETMASK=$MASK" >> /etc/sysconfig/network-scripts/ifcfg-enp0s3
	echo "GATEWAY=$GATE" >> /etc/sysconfig/network-scripts/ifcfg-enp0s3
	%end
