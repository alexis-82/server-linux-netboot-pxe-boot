#### Step 1: Install Required Packages  
```
sudo apt install tftpd-hpa apache2 syslinux isc-dhcp-server
```

#### Step 2: Configure TFTP Server  
```
sudo vim /etc/default/tftpd-hpa
```

Edit the file:  
```
TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/var/lib/tftpboot/amd64"
TFTP_ADDRESS=":69"
TFTP_OPTIONS="--secure --create --listen"
```
:warning: Be aware that in the latest versions, the directory could be /var/lib/tftpboot/amd64 e.g. Ubuntu24
```
sudo mkdir /var/lib/tftpboot
```

#### Step 3: Configure DHCP Server  
```
sudo vim /etc/dhcp/dhcpd.conf
``` 

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
```
sudo systemctl restart isc-dhcp-server
```

#### Step 4: Set Up Netboot Files  
```
wget https://<mirror>-netboot-amd64.tar.gz
```
```
sudo tar -xzvf netboot.tar.gz -C /var/lib/tftpboot/
```
:warning: Mirrors for download file netboot.tar.gz  
Ubuntu only 24.xx:  
```
https://launchpad.net/ubuntu/+cdmirrors
```
Save the changes and restart the TFTP service:  
```
sudo systemctl restart tftpd-hpa
```

#### Step 5: Restart all services
```
sudo systemctl restart isc-dhcp-server
sudo systemctl restart tftpd-hpa
```

#### Step 6: Test PXE Boot  
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
