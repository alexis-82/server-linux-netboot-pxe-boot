#### Step 1: Install Required Packages  
```
sudo apt install tftpd-hpa syslinux isc-dhcp-server
```

#### Step 2: Configure TFTP Server  
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
:warning: Be aware that in the latest versions, the directory could be /var/lib/tftpboot/amd64 e.g. Ubuntu24
```
sudo mkdir /var/lib/tftpboot
```

Save the changes and restart the TFTP service:  
```
sudo systemctl restart tftpd-hpa
```

#### Step 3: Configure DHCP Server  
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

#### Step 4: Set Up Netboot Files  
```
sudo cp /usr/lib/syslinux/modules/bios/* /var/lib/tftpboot/
```
```
wget https://<mirror>-netboot.tar.gz
```
```
sudo tar -xzvf netboot.tar.gz -C /var/lib/tftpboot/
```

:warning: Mirrors for download file netboot.tar.gz  
Ubuntu only 22.xx:  
```
http://archive.ubuntu.com/
```

#### Step 5: Configure PXE Menu
Create a PXE menu  
```
sudo mkdir /var/lib/tftpboot/pxelinux.cfg/
```
```
sudo vim /var/lib/tftpboot/pxelinux.cfg/default
```
Add the following menu structure:  
```
DEFAULT ubuntu
LABEL ubuntu
  MENU LABEL ^Install Ubuntu Server 22
  KERNEL ubuntu-installer/amd64/linux
  APPEND vga=788 initrd=ubuntu-installer/amd64/initrd.gz
```

#### Step 6: Restart all services
```
sudo systemctl restart isc-dhcp-server
sudo systemctl restart tftpd-hpa
```

#### Step 7: Test PXE Boot  
It's time to test the PXE boot process.  
Access the bios and set the first boot to network to bring up the menu for installing the operating system.

#### NOTE
Setting firewall of the system:  
```
sudo ufw allow 69/udp
sudo ufw allow 2049/tcp
sudo ufw allow 111/tcp
sudo ufw allow 111/udp
sudo ufw allow 4011/udp
```