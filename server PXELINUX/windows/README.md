
# Configuring Samba to Serve Windows 11 Installation Files

## 1. PXELINUX Configuration File Setup

Ensure you have correctly configured PXELINUX. You can use a configuration similar to this in the `default` file:

```conf
DEFAULT menu.c32
PROMPT 0
TIMEOUT 50
ONTIMEOUT WinPE

MENU TITLE PXE Boot Menu
LABEL WinPE
    MENU LABEL Boot WinPE
    LINUX memdisk
    INITRD winpe.iso
    APPEND iso raw
```

## 2. Samba Server Configuration

- **Install Samba**:
  ```sh
  apt update
  apt install samba
  ```

- **Configure the share in `/etc/samba/smb.conf`**:
  ```conf
  [win]
  path = /srv/win/
  browsable = yes
  read only = no
  guest ok = yes
  ```

- **Restart the Samba service**:
  ```sh
  systemctl restart smbd
  ```

## 3. Create a Directory for Windows 10/11 Installation Files and Copy the Necessary Files

```sh
mount -o loop Windows.iso /mnt
sudo mkdir -p /srv/win
sudo cp -r /mnt/* /srv/win/
```

## 4. WinPE Client Configuration

- **Boot the WinPE client via PXE**.
- **Initialize the network**:
  ```sh
  wpeinit
  ```
- **Verify network connectivity**:
  ```sh
  ping <server-ip>
  ```
- **Mount the network share with `net use`**:
  ```sh
  net use Z: \\<server-ip>\win
  ```
- **Run the Windows installation program**:
  ```sh
  Z:\setup.exe
  ```

## Recommendations

- **Verify Samba Permissions**: Ensure that the permissions on the Samba server allow users to read the necessary files. You can make the share accessible to everyone (guest access) for simplicity, but be cautious about security.
  ```sh
  sudo chmod -R 755 /srv/win/
  ```

- **Network Settings**: Ensure that network settings are correct and that there are no firewalls blocking SMB/CIFS access on the network.

- **Log Monitoring**: Monitor Samba logs to diagnose any issues. Log files can typically be found in `/var/log/samba/`.
  ```sh
  sudo tail -f /var/log/samba/log.smbd
  ```