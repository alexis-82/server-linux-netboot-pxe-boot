# Steps to Configure PXE Boot with APACHE

## Step 1: Install Required Packages  
```
sudo apt install tftpd-hpa apache2 syslinux pxelinux isc-dhcp-server
```

## Step 2: Configure TFTP Server  
```
sudo vim /etc/default/tftpd-hpa
```

Edit the file:  
```
TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/var/lib/tftpboot"
TFTP_ADDRESS=":69"
TFTP_OPTIONS="--secure --create --listen"
```

```
sudo mkdir /var/lib/tftpboot
```


## Step 3: Configure DHCP Server  
```
sudo vim /etc/dhcp/dhcpd.conf
``` 

Add the following to the configuration file:  
```
# Uncomment this line if you are using your local network
authoritative;

# A slightly different configuration for an internal subnet.
subnet <network-address> netmask <subnet-mask> {
  range <ip-range-start> <ip-range-end>;
  option subnet-mask <subnet-mask>;
  option domain-name-servers <address DNS router o eg. Google>; 
  #option domain-name "internal.example.org";
  option routers <gateway-address>;
  option broadcast-address <xxx.xxx.x.255>;
  next-server <tftp-server-ip>;
  filename "pxelinux.0";
  default-lease-time 600;
  max-lease-time 7200;
}
```
## Step 4: Mount ISO  
```
mount -o loop /home/$USER/ubuntu-24.04-live-server-amd64.iso /mnt/
```
```
mkdir /var/lib/tftpboot/ubuntu-server
```
```
cp -rf /mnt/. /var/lib/tftpboot/ubuntu-server/
```
```
chmod -R 755 /var/lib/tftpboot/ubuntu-server/
```

## Step 5: Set Up Apache  

```
cd /var/www/html/
```
```
mkdir ubuntu-server
```
```
cd ubuntu-server
```
```
cp /home/$USER/ubuntu-24.04-live-server-amd64.iso .
```
```
cd /var
```
```
chown -R www-data:www-data www/
```

## Step 6: Files boot  
```
cp /usr/lib/syslinux/memdisk /var/lib/tftpboot
```

BIOS
```
cp /usr/lib/syslinux/modules/bios/{chain.c32,ldlinux.c32,libcom32.c32,libutil.c32,mboot.c32,menu.c32,vesamenu.c32} /var/lib/tftpboot
```

EFI64
```
cp /usr/lib/syslinux/modules/efi64/{ldlinux.e64,chain.c32,libcom32.c32,libutil.c32,mboot.c32,menu.c32,vesamenu.c32} /var/lib/tftpboot
```

## Step 7: Configure PXE Menu
Create a PXE menu  
```
sudo mkdir /var/lib/tftpboot/pxelinux.cfg/
```
```
sudo vim /var/lib/tftpboot/pxelinux.cfg/default
```
Add the following menu structure:  
```
default menu.c32
prompt 0
timeout 300
ONTIMEOUT local

menu title ############ PXE Boot Menu ##############
label 1
menu label ^1) Install Ubuntu Server
kernel ubuntu-server/casper/vmlinuz
append boot=casper initrd=ubuntu-server/casper/initrd ip=dhcp url=http://192.168.1.150/ubuntu-server/ubuntu.iso live-media=/casper/ ignore_uuid ---

label 2
menu label ^2) Boot from local drive 
localboot 0
```

## Step 8: Restart all services
```
sudo systemctl restart isc-dhcp-server
sudo systemctl restart tftpd-hpa
sudo systemctl restart apache2
```

## Step 9: Test PXE Boot  
It's time to test the PXE boot process.  
Access the bios and set the first boot to network to bring up the menu for installing the operating system.

## NOTE
Setting firewall of the system:  
```
sudo ufw allow 69/udp
sudo ufw allow 2049/tcp
sudo ufw allow 111/tcp
sudo ufw allow 111/udp
sudo ufw allow 4011/udp
```