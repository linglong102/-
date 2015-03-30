---
layout: post
title: kickstart 自动化部署服务器端环境安装脚本
category: 自动化部署
tags: Pxe+kickstart
keywords: kickstart 自动化部署服务器端环境安装脚本
description: 
---

>**Data:2015/3/29 6:02:27**

	#!/bin/bash
	#
	echo "Author: Jerrybaby"
	echo "https://github.com/Jerrybaby"
	echo "License: GNU GENERAL PUBLIC LICENSE"
	echo "
          ------------------------------------------------------------
          ------------------------------------------------------------
          ------------------------------------------------------------
          ------- kickstart 自动化部署服务器端环境安装脚本 -----------
          ------------------------------------------------------------
          ------------------------------------------------------------
          ------------------------------------------------------------
		 "

	# environment
	setenforce 0 &> /dev/null
	service iptables stop &> /dev/null

	if [ -d "/var/lib/tftpboot" ]
	then
    	rm -rf /var/lib/tftpboot/*
    	rm -rf /tftpboot
	fi
	if [ -f "/var/www/html/files/ks.cfg" ]
	then
	    rm -rf /var/www/html/files/*
	fi
		
	IFACE=`ping -I eth0  -c 4 baidu.com | grep Unreachable | wc -l`
	if [ -z $IFACE ]
	then
	    eth=eth1
	else
	    eth=eth0
	fi
	IPADDR=`ifconfig $eth | grep 'inet addr' | awk -F[:' ']+ '{print $4}'`

	download_ubuntu ()
	{
		echo "Downloading ubuntu-14.04.2-server-amd64......"
	#	wget http://119.255.9.54/cdimage.ubuntu.com/releases/14.04.1/release/	ubuntu-14.04.2-server-amd64+mac.iso
		mount -o loop /root/ubuntu-14.04-server-amd64.iso /mnt
		echo "OK!"
	}

	download_centos ()
	{
	    echo "Downloading centos 6.6-x86_64-minimal......"
	#   wget http://mirrors.sohu.com/centos/6/isos/x86_64/CentOS-6.6-x86_64-minimal.iso
	    mount -o loop /root/CentOS-6.5-x86_64-bin-DVD1.iso /mnt
	#    mount -o loop /root/CentOS-7.0-1406-x86_64-DVD.iso /mnt
	    echo "OK!"
	}

	install_packages ()
	{
	    echo "Installing packages......"
	    yum -y install httpd dhcp tftp-server syslinux
	    echo "OK!"
	}

	conf_ubuntu_files ()
	{
		echo "Configuring ubuntu files......"
		mkdir -p /var/www/html/files
		mkdir -p /var/www/html/ubuntu
		cp -rf /mnt/* /var/www/html/files
		ln -s /var/lib/tftpboot /tftpboot
		cp -rf /mnt/install/netboot/* /tftpboot
		sed -i "s@\(^[[:space:]]append\) vga=788@\1 ks=http://${IPADDR}/files/ks.cfg preseed/url=http://${IPADDR}/files/preseed/ubuntu-server.seed vga=788@g" /tftpboot/ubuntu-installer/amd64/boot-screens/txt.cfg
		echo "d-i     live-installer/net-image string http://${IPADDR}/files/install/filesystem.squashfs" | tee -a /var/www/html/files/preseed/ubuntu-server.seed
		sed -i 's/^timeout 0/timeout 1/g' /tftpboot/ubuntu-installer/amd64/boot-screens/syslinux.cfg
		echo "d-i mirror/protocol string http" | tee -a /var/www/html/files/preseed/ubuntu-server.seed
		echo "d-i mirror/http/hostname string cn.archive.ubuntu.com" | tee -a /var/www/html/files/preseed/ubuntu-server.seed
		echo "d-i mirror/http/directory string /ubuntu" | tee -a /var/www/html/files/preseed/ubuntu-server.seed
	}

	conf_centos_files ()
	{
	    echo "Configuring centos files......"
	    mkdir -p /var/www/html/files
		cp -rf /mnt/* /var/www/html/files
	    ln -s /var/lib/tftpboot /tftpboot
	    mkdir -p /tftpboot/pxelinux.cfg
	    cp /usr/share/syslinux/pxelinux.0 /tftpboot
	    cp /var/www/html/files/images/pxeboot/initrd.img /tftpboot
	    cp /var/www/html/files/images/pxeboot/vmlinuz /tftpboot
	    cp /var/www/html/files/isolinux/*.msg /tftpboot
	    cp /var/www/html/files/isolinux/isolinux.cfg /tftpboot/pxelinux.cfg/default
	    chmod 777 /tftpboot/pxelinux.cfg/default
	    sed -i 's/\(^[[:space:]]disable.*\)yes$/\1no/g' /etc/xinetd.d/tftp
	    sed -i "s/default.*32$/default linux ks=http:\/\/${IPADDR}\/files\/ks.cfg ksdevice=eth0/g" /tftpboot/pxelinux.cfg/default
	    sed -i 's/timeout 600/timeout 1/g' /tftpboot/pxelinux.cfg/default
	    echo "OK!"
	}

	dhcp_pro ()
	{
	    echo "配置 DHCP...."
	    (cat | tee /etc/dhcp/dhcpd.conf) << EOF
	ddns-update-style interim;
	ignore client-updates;
	next-server $IPADDR;
	filename "/pxelinux.0";
	subnet 172.16.15.0 netmask 255.255.255.0 {
	   option routers          172.16.15.1;
	   option subnet-mask      255.255.255.0;
	   option domain-name-servers      172.16.254.21, 172.16.254.22;
	   option time-offset      -18000;
	   range dynamic-bootp     172.16.15.100 172.16.15.200;
	   default-lease-time 21600;
	   max-lease-time 43200;
	}
	EOF
	    echo "OK!"
	}

	ks_cfg__ubuntu ()
	{
		echo "配置 ks.cfg ...."
	    (cat | tee /var/www/html/files/ks.cfg) << EOF
	#platform=x86, AMD64, or Intel EM64T
	#version=DEVEL
	# Firewall configuration
	firewall --disabled
	# Install OS instead of upgrade
	install
	# Use network installation
	url --url="http://$IPADDR/files"
	# Root password
	#rootpw --plaintext nishishadan
	rootpw --disabled
	user jerry --fullname="jerry" --password nishishadan
	# System authorization information
	auth  --useshadow  --passalgo=sha512
	# Use text mode install
	text
	# System keyboard
	keyboard us
	# System language
	lang en_US
	# Do not configure the X Window System
	skipx
	# Installation logging level
	logging --level=info
	# Reboot after installation
	reboot
	# System timezone
	timezone  Asia/Shanghai
	# Network information
	network  --bootproto=dhcp --noipv6
	# System bootloader configuration
	bootloader --location=mbr
	# Clear the Master Boot Record
	zerombr
	# Partition clearing information
	clearpart --all --initlabel
	# Disk partitioning information
	part /boot --fstype="ext4" --size=200
	part / --fstype="ext4" --size=15000
	part swap --fstype="swap" --size=1024

	%post --interpreter /bin/bash

	apt-get -y install ssh

	echo deb http://ppa.launchpad.net/saltstack/salt/ubuntu `lsb_release -sc` main | sudo tee /etc/apt/sources.list.d/saltstack.list
	wget -q -O- "http://keyserver.ubuntu.com:11371/pks/lookup?op=get&search=0x4759FA960E27C0A6" | sudo apt-key add -
	apt-get update
	apt-get install salt-minion
	echo "master: $IPADDR" >> /etc/salt/minion
	service salt-minion restart

	# set the file limit
	ulimit -SHn 65535
	echo "*    soft    nofile    60000" >> /etc/security/limits.conf
	echo "*    hard    nofile    65535" >> /etc/security/limits.conf

	%end
	EOF
	    echo "OK!"
	}

	ks_cfg_centos ()
	{
	    echo "配置 ks.cfg ...."
	    (cat | tee /var/www/html/files/ks.cfg) << EOF
	#platform=x86, AMD64, or Intel EM64T
	#version=DEVEL
	# Firewall configuration
	firewall --disabled
	# Install OS instead of upgrade
	install
	# Use network installation
	url --url="http://$IPADDR/files"
	# Root password
	rootpw --plaintext nishishadan
	# System authorization information
	auth  --useshadow  --passalgo=sha512
	# Use text mode install
	text
	# System keyboard
	keyboard us
	# System language
	lang en_US
	# SELinux configuration
	selinux --disabled
	# Do not configure the X Window System
	skipx
	# Installation logging level
	logging --level=info
	# Reboot after installation
	reboot
	# System timezone
	timezone  Asia/Shanghai
	# Network information
	network  --bootproto=dhcp --onboot=on --noipv6
	# System bootloader configuration
	bootloader --location=mbr
	# Clear the Master Boot Record
	zerombr
	# Partition clearing information
	clearpart --all --initlabel
	# Disk partitioning information
	part /boot --fstype="ext4" --size=200
	part / --fstype="ext4" --size=151954
	part swap --fstype="swap" --size=4096

	%packages
	@base
	@console-internet
	@core
	@debugging
	@directory-client
	@hardware-monitoring
	@large-systems
	@network-file-system-client
	@performance
	@perl-runtime
	@server-platform
	@server-policy
	@workstation-policy
	pax
	oddjob
	sgpio
	device-mapper-persistent-data
	samba-winbind
	certmonger
	pam_krb5
	krb5-workstation
	perl-DBD-SQLite
	%end

	%post --interpreter /bin/bash
	IP=`ifconfig em1 | grep inet | grep -v inet6 | awk -F[' ':]+ '{print $4}'`
	MASK=`ifconfig em1 | grep inet | grep -v inet6 | awk -F[' ':]+ '{print $8}'`
	GATE=`route -n | grep UG | awk '{print $2}'`
	sed -i "s@\(^H.*=\).*@\1server-$IP@g" /etc/sysconfig/network
	sed -i "s@\(^BOOT.*=\).*@\1static@g" /etc/sysconfig/network-scripts/ifcfg-em1
	echo "IPADDR=$IP" >> /etc/sysconfig/network-scripts/ifcfg-em1
	echo "NETMASK=$MASK" >> /etc/sysconfig/network-scripts/ifcfg-em1
	echo "GATEWAY=$GATE" >> /etc/sysconfig/network-scripts/ifcfg-em1
	%end
	EOF
	    echo "OK!"
	}

	rest_ser ()
	{
	    echo "重启服务...."
	    umount /mnt
	    service httpd restart
	    service xinetd restart
	    service dhcpd restart
	    chkconfig httpd on
	    chkconfig dhcpd on
	    chkconfig xinetd on
	    chkconfig tftp on
	    echo "OK!"
	}

	read -p "本机 IP: $IPADDR,是否正确？Y/N" para
	case $para in
	Y|y)
		read -p "请选择 Ubuntu(U) 或 CentOS(C)" sys
		case $sys in
		U|u)
			download_ubuntu
			install_packages
			conf_ubuntu_files
			dhcp_pro
			ks_cfg__ubuntu
			rest_ser
			;;
		C|c)
			download_centos
			install_packages
			conf_centos_files
			dhcp_pro
			ks_cfg_centos
			rest_ser
			;;
		esac
		;;
	N|n)
	    echo "请修改正确的 IP 地址"
	    exit
	    ;;
	esac
