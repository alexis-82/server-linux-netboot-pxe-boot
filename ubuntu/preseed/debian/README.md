# Using Preseed for Automated Debian Installation

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
```
label install
    menu label ^Install Debian
    kernel debian/debian-installer/amd64/linux
    append initrd=debian/debian-installer/amd64/initrd.gz root=/dev/nfs nfsroot=192.168.1.153:/srv/nfs/debian netboot=nfs ip=dhcp rw nfsroot auto=true priority=critical preseed/url=tftp://192.168.1.153/debian/preseed.cfg
```
:warning: preseed/url is of the server TFTP: /var/lib/tftpboot/debian

3. During network boot, the PXE client will download the `preseed.cfg` file and use it to automatically answer questions during the Ubuntu installation process.

Using `preseed.cfg` allows you to fully automate the Debian installation, reducing the risk of errors and simplifying the installation process across multiple machines.

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