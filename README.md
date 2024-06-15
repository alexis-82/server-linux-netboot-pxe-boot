# Server linux netboot with pxe-boot
---
:it:
### Una guida completa passo-passo per configurare un ambiente di netboot utilizzando il protocollo PXE (Pre-boot Execution Environment) su server Linux, consentendo l'avvio di sistemi operativi e ambienti live direttamente dalla rete senza la necessità di supporti di archiviazione locali.

:uk:
### A comprehensive step-by-step guide to setting up a netboot environment using the PXE (Pre-boot Execution Environment) protocol on Linux server, allowing you to boot operating systems and live environments directly from the network without the need for local storage media.

#### Step 1: Install Required Packages  
```sudo apt install tftpd-hpa apache2 syslinux isc-dhcp-server```

#### Step 2: Configure TFTP Server  
```sudo vim /etc/default/tftpd-hpa```

Edit the file:  
```
TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/var/lib/tftpboot"
TFTP_ADDRESS=":69"
TFTP_OPTIONS="--secure"
```
:warning: Be aware that in the latest versions, the directory could be /var/lib/tftpboot/amd64 e.g. Ubuntu24

Save the changes and restart the TFTP service:  
```sudo systemctl restart tftpd-hpa```

#### Step 3: Configure DHCP Server  
```sudo vim /etc/dhcp/dhcpd.conf``` 

Add the following to the configuration file:  
```
subnet <network-address> netmask <subnet-mask> {
  range <ip-range-start> <ip-range-end>;
  option subnet-mask <subnet-mask>;
  option domain-name-servers <address DNS router o eg. Google>; 
  option routers <gateway-address>;
  next-server <tftp-server-ip>;
  filename "pxelinux.0";
}
```
Save the changes and restart the DHCP service:  
```sudo systemctl restart isc-dhcp-server```

#### Step 4: Set Up Netboot Files  
```sudo cp /usr/lib/syslinux/modules/bios/* /var/lib/tftpboot/```
```wget http://archive.ubuntu.com/ubuntu/dists/stable/main/installer-amd64/current/legacy-images/netboot/netboot.tar.gz```
```sudo tar -xzvf netboot.tar.gz -C /var/lib/tftpboot/```
or /var/lib/tftpboot/amd64   

:warning: Mirrors for download file netboot.tar.gz  
Debian:  
```https://www.debian.org/mirror/list.it.html```
Ubuntu:  
```https://launchpad.net/ubuntu/+cdmirrors```

#### Step 5: Configure PXE Menu
Create a PXE menu  
```sudo vim /var/lib/tftpboot/pxelinux.cfg/default```
Add the following menu structure:  
```
DEFAULT ubuntu
LABEL ubuntu
  MENU LABEL ^Install Ubuntu Server 22
  KERNEL ubuntu-installer/amd64/linux
  APPEND vga=788 initrd=ubuntu-installer/amd64/initrd.gz
```
From file.iso:  
```APPEND ISO initrd=ubuntu-22.04-live-server-amd64.iso raw```

From URL:  
```APPEND root=/dev/ram0 ramdisk_size=1500000 ip=dhcp url=http://releases.ubuntu.com/focal/ubuntu-22.04.1-live-server-amd64.iso```

:warning: For Ubuntu 24 is not necessary, everything is already configured!  

#### Step 6: Test PXE Boot  
It's time to test the PXE boot process.  
Access the bios and set the first boot to network to bring up the menu for installing the operating system.