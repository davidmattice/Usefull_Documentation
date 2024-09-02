# Proxmox Virtual Environment Setup

Proxmox Virtual Environment - Home Lab Setup

## Proxmox Virtual Environment Installation

Download Proxmox from [here](https://www.proxmox.com/en/proxmox-virtual-environment/get-started) and follow the instructions to write to an ISO or USB boot drive

## Proxmox Virtual Environment Host Updates

This should all be in a script that can be run.

### Switch from static IP to DHCP with Reservation

Update the *vmbr0* interface to be dhcp - update: /etc/network/interfaces
```
sed -i '/^iface vmbr0 inet static/{s/static/dhcp/;n;s/^/#/;n;s/^/#/}' ifs
```

Setup the auto update of the hosts file - create: /etc/dhcp/dhclient-exit-hooks.d/update-etc-hosts
```
cat << HERE_DOC >/etc/dhcp/dhclient-exit-hooks.d/update-etc-hosts
# Set some useful variables
hn=\$(hostname)
hnf=\$(hostname -f)
file="/etc/hosts"

# If required update the file
if ([ \$reason = "BOUND" ] || [ \$reason = "RENEW" ]); then
  sed -i "s/^.*\s\${hnf}\s.*\$/\${new_ip_address} \${hnf} \${hn}/" \${file}
fi
HERE_DOC
```
Reference: [Proxmox forum article](https://forum.proxmox.com/threads/set-a-dynamic-address-to-pve.119847/) and [related link](https://weblog.lkiesow.de/20220223-proxmox-test-machine-self-servic/proxmox-server-dhcp.html)

### Remove subscription message
```
sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service
```

### Configure package updates for use with no subscription
```
mv /etc/apt/sources.list.d/ceph.list /etc/apt/sources.list.d/ceph.orig
echo "deb http://download.proxmox.com/debian/ceph-quincy bookworm no-subscription" >/etc/apt/sources.list.d/ceph.list
echo "deb http://download.proxmox.com/debian/ceph-reef bookworm no-subscription" >>/etc/apt/sources.list.d/ceph.list
mv /etc/apt/sources.list.d/pve-enterprise.list /etc/apt/sources.list.d/pve-enterprise.orig
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" >/etc/apt/sources.list.d/pve.list
```

### Add support for snippets
```
pvesm set local --content backup,iso,vztmpl,snippets
chmod 777 /var/lib/vz/snippets
```

## Configure custom user for Terraform access (THIS IS NOT WORKING)

### Add a user and role for Terraform to use
```
export TF_PASSWORD="PASSWORD_HERE"
export TF_USERNAME="f-terraform"
pveum role add terraform-role -privs "VM.Allocate VM.Clone VM.Config.CDROM VM.Config.CPU VM.Config.Cloudinit VM.Config.Disk VM.Config.HWType VM.Config.Memory VM.Config.Network VM.Config.Options VM.Monitor VM.Audit VM.PowerMgmt Datastore.AllocateSpace Datastore.Audit Sys.Audit Datastore.Allocate SDN.Use Sys.Modify"
pveum user add ${TF_USERNAME}@pve -password ${TF_PASSWORD} -comment "Terraform Functional User"
pveum aclmod / -user ${TF_USERNAME}@pve -role terraform-role
useradd -m -s /bin/bash ${TF_USERNAME}
echo "export PATH=$PATH:/usr/sbin" >>~${TF_USERNAME}/.bashrc
passwd ${TF_USERNAME}
```
Information on using Terraform is [here](https://github.com/davidmattice/Usefull_Documentation/blob/main/terraform/README.md)

## Some Optional items

### Add 2nd NIC as a bridge to another network (not tested)

Add a 2nd bridge for private networking (Available if 2nd NIC is installed)
```
cat >>/etc/networkinterfaces <<<EOF
auto vmbr1
iface vmbr1 inet manual
        bridge-ports none
        bridge-stp off
        bridge-fd 0
EOF
ifreload -a
```

### Disable IPv6
```
cat >>/etc/sysctl.d/disable-ipv6.conf <<<EOF
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
EOF
```



## References

### The BPG Proxmox Terraform Provider

- [BPG Proxmox Provider](https://registry.terraform.io/providers/bpg/proxmox/latest/docs)
- [BPG Examples](https://github.com/bpg/terraform-provider-proxmox/tree/main/example)

### Reference repositories

- [My home operations repo using K8](https://github.com/szinn/k8s-homelab)
- [Small Home Lab for K8 using PVE](https://github.com/ehlesp/smallab-k8s-pve-guide)

### Secondary References

- [Cloud Init Template Builder Script](https://gist.github.com/chris2k20)
- [Cloud-Init file reference](https://github.com/chris2k20/proxmox-cloud-init/tree/main)
- [ProxMox Cloud Init Examples](https://dustinrue.com/2020/05/going-deeper-with-proxmox-cloud-init/)
