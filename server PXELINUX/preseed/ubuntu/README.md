# Using Preseed for Automated Ubuntu Installation

The `preseed.cfg` file allows you to automate the Ubuntu installation process, reducing or completely eliminating the need for manual interaction. This file contains the default answers to the questions asked during installation, allowing for an unattended, hands-off installation.

## Preparing the preseed.cfg file
1. Create a new text file and name it `preseed.cfg`.
2. Define the desired settings in the file, such as:
   - Language and keyboard
   - Network configuration 
   - Package download mirror
   - User account settings
   - Disk partitioning
   - Package selection to install

## Using the preseed.cfg file during installation
1. Copy the `preseed.cfg` file to the TFTP export directory on the server.
2. Modify the PXE Linux configuration file (usually `pxelinux.cfg/default`) to include the necessary parameters to use the `preseed.cfg` file. For example:

### NFS + TFTP
```
label install
    menu label ^Install Ubuntu
    kernel ubuntu/ubuntu-installer/amd64/linux
    append initrd=ubuntu/ubuntu-installer/amd64/initrd.gz root=/dev/nfs nfsroot=192.168.1.153:/srv/nfs/debian netboot=nfs ip=dhcp rw nfsroot auto=true priority=critical preseed/url=tftp://192.168.1.153/ubuntu/preseed.cfg
```

### NFS + APACHE
```
append initrd=ubuntu/ubuntu-installer/amd64/initrd.gz root=/dev/nfs nfsroot=192.168.1.153:/srv/nfs/ubuntu netboot=nfs ip=dhcp rw auto=true priority=critical preseed/url=http://192.168.1.153/ubuntu/preseed.cfg
```

### APACHE
```
append initrd=ubuntu/ubuntu-installer/amd64/initrd.gz root=/dev/ram0 ramdisk_size=1500000 ip=dhcp url=http://192.168.1.153/ubuntu/ubuntu.iso auto=true priority=critical preseed/url=http://192.168.1.153/ubuntu/preseed.cfg
```

:warning: preseed/url is of the server TFTP: /var/lib/tftpboot/ubuntu

3. During network boot, the PXE client will download the `preseed.cfg` file and use it to automatically answer questions during the Ubuntu installation process.

Using `preseed.cfg` allows you to fully automate the Ubuntu installation, reducing the risk of errors and simplifying the installation process across multiple machines.

## Generate password for User
### With mkpasswd:

```
apt install whois
```
For generate:
```
printf "PASSWORD" | mkpasswd -s -m md5
```
### With OpenSSL:
```
printf 'PASSWORD' | openssl passwd -6 -salt 'FhcddHFVZ7ABA4Gi' -stdin
```
or
```
perl -e 'print crypt($ARGV[0], "\$6\$saltstringcryptica") . "\n"' PASSWORD
```