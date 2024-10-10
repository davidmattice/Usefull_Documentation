# Creating a Windows 11 VM - as of 2024/10/5

##  Download ISO images and make available

Windows 11 Image can be downloaded from the [Microsoft site](https://www.microsoft.com/en-us/software-download/windows11)

## Optional: Remove "Press Any Key" prompt (Not working with UEFI BIOS)

To remove the prompt we will have to create an updated ISO image using [Anyburn](https://www.anyburn.com/download.php)

### References

- Video to edit an [ISO with anyburn](https://www.google.com/search?q=best+way+to+add+files+to+windows+11+iso&sca_esv=4614768026efb3bf&sxsrf=ADLYWIJDTqLM31Sl0GKDer0inVc2DNIRbw%3A1728323621695&ei=JSAEZ5WZKv-MwbkP3_XmkAU&ved=0ahUKEwjVv5yc6_yIAxV_RjABHd-6GVIQ4dUDCA8&uact=5&oq=best+way+to+add+files+to+windows+11+iso&gs_lp=Egxnd3Mtd2l6LXNlcnAiJ2Jlc3Qgd2F5IHRvIGFkZCBmaWxlcyB0byB3aW5kb3dzIDExIGlzbzIIEAAYgAQYogQyCBAAGIAEGKIEMggQABiABBiiBDIIEAAYgAQYogQyCBAAGIAEGKIESKQ4ULIFWKwpcAF4AZABAJgBywGgAdkLqgEFMi45LjG4AQPIAQD4AQGYAgygArAKwgIKEAAYsAMY1gQYR8ICCBAhGKABGMMEwgIKECEYoAEYwwQYCpgDAIgGAZAGCJIHBDIuMTCgB6Qp&sclient=gws-wiz-serp#fpstate=ive&vld=cid:b8c2e392,vid:RRUmTxOX1KE,st:0)
- Details on what to [edit in the ISO](https://scadminsblog.wordpress.com/2017/05/18/how-to-remove-the-message-press-any-key-to-boot-from-cd-or-dvd-with-powershell/)

## Create VM

1. General
    1. *VM ID: Set the VM ID*
    2. *Name: Set the name of the VM*
    3. **Advanced:** *Start at boot: Enabled*
2. OS
    1. Storage: local
    2. *ISO image: Select the ISO for Windows from above*
    3. *Guest OS Type: Wicrosoft Windows*
    4. *Version: 11/2022/2025*
    5. *Add Additional Drive for VirtIO: Enable*
    6. Storage: local
    7. *ISO image: Select the ISO from VirtIO from above*
3. System
    1. Graphics Card: Default
    2. Machine: q35
    3. BIOS: OVMF (UEFI)
    4. *EFI Storage: local-lvm*
    5. *SCSI Controller: Virto SCSI*
    6. *Qemu Agent: Enabled*
    7. *TPM Storage: local=lvm*
    8. Version: v2.0
4. Disk
    1. *Bus/Device: Virtio Block*
    2. Storage: local-lvm
    3. *Disk Size (GiB): 60 (min)*
    4. Cache: Default
5. CPU
    1. Sockets: 1
    2. *Cores: 2 (or whatever)*
    3. *Type: Host (to be able to enable WSL)*
6. Memory
    1. *Memory (MiB): 4096 (or whatever)*
7. Network
    1. Bridge: vmbr0
    2. VLAN Tag: no VLAN
    3. *Model: VirtIO (paravirtualized)*
    4. MAC address: auto


## Boot

Open a noVNC console, and select start and then press any key to boot from CD

## Installation

When the install gets to the hard drive and it is not there, add drivers for disk & network ...

1. Select *load driver*
2. Select *Browse*
3. Expand CD with *vrtio*
    1. Expand *amd64*
    2. Select *w11*
    3. Click *OK*
    4. Click *Next*
4. Select *load driver*
5. Select *Browse*
6. Expand CD with *vrtio*
    1. Expand *netKVM/w11*
    2. Select *amd64*
    3. Click *OK*
    4. Click *Next*

Continue the installation

[See Proxmox Windows 10 Best Pratices](https://pve.proxmox.com/wiki/Windows_10_guest_best_practices)

## Optional: Disable IPv6 on the Widnows VM (no need for it)

1. Go to settings
2. Network & Internet
3. Advanced Netowrk Settings
4. Ethernet
5. More Adapter Options (Edit)
6. Disable IPv6 (uncheck)
7. Click OK

## Install QEMU drivers on the Windows VM

1. Open *Device Manager*
2. Find "PCI Simple Communications Controller"
3. Right Click and select *Update Driver*
4. Select *Browse* and *Browse* again
5. Expand CD with *virtio*
    1. Expand *vioserial/w11*
    2. Select *amd64*
    3. Click *OK*
    4. Click *Next*
6. Open Explorer
7. Expand CD with *virtio*
    1. Expand *guest-agent*
    2. Double click on *qemu-ga-x86_64*
    3. Click *Yes*
8. Shutdown the VM - **a reboot will not work**

[See Proxmox Qemu Guest Agent](https://pve.proxmox.com/wiki/Qemu-guest-agent)

## Enable VNC on the VM

1. Open a shell to the PVE Host
2. Update /etc/pve/local/qemu-server/<VMID>.conf
3. Add
```
args: -vnc 0.0.0.0:<Display port> # Display port can be any 2 digit unique number, higher is beeter
```
4. Start the VM

Connect from VNC Viewer using the PVE Host IP and the Display port: "**\<IP\>:\<port\>**"

[See Proxmox VNC Client Access](https://pve.proxmox.com/wiki/VNC_Client_Access)

## Optional: Install/Setup WSL2

1. Open a Command prompt as Administrator
```
wsl --install
```
2. Reboot
3. Enter a username and password
