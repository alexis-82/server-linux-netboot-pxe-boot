# Steps to Configure PXE Boot with NFS

## 1. Download Debian Netboot Files
Download the necessary files for Debian netboot, which include the kernel and initrd.
```bash
wget http://ftp.debian.org/debian/dists/stable/main/installer-amd64/current/images/netboot/netboot.tar.gz -O /var/lib/tftpboot/debian/netboot.tar.gz
```

## 2. Extract Netboot Files
Extract the netboot files into the TFTP directory:
```bash
cd /var/lib/tftpboot/debian/
sudo tar -xvzf netboot.tar.gz
```

## 3. Configure `pxelinux.cfg/default`
Edit or create the PXE configuration file:
```bash
sudo mkdir -p /var/lib/tftpboot/pxelinux.cfg
sudo vim /var/lib/tftpboot/pxelinux.cfg/default
```
Add or update the file content as follows:
```plaintext
DEFAULT menu.c32
PROMPT 0
TIMEOUT 300
ONTIMEOUT local
MENU TITLE PXE Boot Menu

LABEL install
    MENU LABEL Install Debian
    NETBOOT KERNEL debian/debian-installer/amd64/linux
    APPEND initrd=debian/debian-installer/amd64/initrd.gz root=/dev/nfs nfsroot=192.168.1.150:/srv/nfs/debian netboot=nfs ip=dhcp rw
```

## 4. Configure the NFS Server
Ensure the NFS server is installed and properly configured. Install the NFS server:
```bash
sudo apt update
sudo apt install nfs-kernel-server
```
Create the NFS directory for Debian and copy the installation files:
```bash
sudo mkdir -p /srv/nfs/debian
cd /srv/nfs/debian
sudo cp -r /var/lib/tftpboot/debian/debian-installer /srv/nfs/debian
```
Edit the `/etc/exports` file to export the NFS directory:
```bash
sudo vim /etc/exports
```
Add the following line:
```plaintext
/srv/nfs/debian 192.168.1.0/24(ro,sync,no_subtree_check)
```
Export the directories:
```bash
sudo exportfs -a
```
Restart the NFS server to apply the changes:
```bash
sudo systemctl restart nfs-kernel-server
```

## 5. Configure the DHCP Server
Ensure the DHCP server configuration file (`/etc/dhcp/dhcpd.conf`) is properly set up to provide the necessary files for PXE boot:
```bash
sudo nano /etc/dhcp/dhcpd.conf
```
Update the content as follows, adjusting for your network:
```plaintext
default-lease-time 600;
max-lease-time 7200;
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.50 192.168.1.100;
    option routers 192.168.1.1;
    option domain-name-servers 8.8.8.8;
    option subnet-mask 255.255.255.0;
    # PXE Boot options
    filename "pxelinux.0";
    next-server 192.168.1.150;
}
```

## 6. Configure the TFTP Server
Ensure the TFTP server is properly configured to serve the netboot files. If using `tftpd-hpa`, the configuration file should look like this:
```bash
sudo nano /etc/default/tftpd-hpa
```
Ensure it contains:
```plaintext
TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/var/lib/tftpboot/"
TFTP_ADDRESS="0.0.0.0:69"
TFTP_OPTIONS="--secure --create --listen"
```
Restart the TFTP service to apply the changes:
```bash
sudo systemctl restart tftpd-hpa
```

## 7. Test PXE Boot
Restart a client configured for network boot (PXE) and verify that it can boot correctly and find the necessary files for the Debian installation.

## Debug and Logs
If you encounter issues, check the logs for more details:
```bash
sudo journalctl -u tftpd-hpa
sudo journalctl -u nfs-kernel-server
sudo journalctl -u isc-dhcp-server
```

### Final Note
With these steps, your PXE client should boot and correctly find the necessary files for the Debian installation via netboot using NFS. If you have further issues or questions, let me know!