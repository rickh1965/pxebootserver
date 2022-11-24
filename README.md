# Customize all automated installations of Rocky Linux and Proxmox Servers in your environment

The point of this entire excercise is to have a way to boot preset configurations for your server automatically from with PXE. Yes, this is cobbler-like. This is super simple and just asks for a few parameters in a text file and ansible. Foreman, MAAS (no Debian support), Ironic and other tools do this. They are slick and do this in an automated fashion. If you're not interested in loading new software, this is a minimal i.e "poor man's solution" to bare-metal provisioning. 

As a bonus, if you are running an old HP server, I've included a playbook which puts your Perc 410i (probably works on others) in HBA mode for Ceph/ZFS operations. Those instructions are below.

Create PXE Boot Server from Rocky Linux host
=========

This was taken (mostly) from these very nice instructions on setting up a PXE boot server.
https://www.linuxhelp.com/how-to-configure-pxe-network-boot-server-for-multiple-linux-distribution

Easy to follow and reasonably easy to adapt to Rocky and create playbooks from.

This Playbook creates a PXE boot server that handles DHCP and booting. The default menu contains an interactive install of Rocky Linux 8 as well as Debian 11. Also an automated installation of each with DHCP. Future playbooks will leverage kickstart scripts to reliably build your environment from scratch.

Requirements
------------

A plain Rocky 8 server with a static IP and network connectivity. I have tested this on a server created with my make_bootiso playbook on this site. This will install dnsmasq and vsftp to serve the install media via ftp. The timeout option will boot from local disk. Setting your servers to default boot via network and without interaction your server will boot its installed OS by default.

Role Variables
--------------

To install your very own PXE boot server, make sure that your router  or anything else is not already serving DNS on the network you will install this server on.  You will need the following information about you site to build this PXE Boot Server. The SSH_KEY variable will allow the user on your bastion server to directly log into the newly configured host and will automatically be allowed passwordless sudo privledges. Do not define this value if that is not wanted.

This file is located in 
``` roles/pxeboot/defaults/main.yml```

```yaml
#################### dnsmasq variables

network_interface: "enp1s0"
dhcp_domain: "home.lan"
dhcp_gw: "192.168.1.1"
dhcp_range: "192.168.1.100,192.168.1.200"
netmask: "255.255.255.0"
lease_time: "1h"
pxe_server_ip: "192.168.1.2"
dns_ip: "1.1.1.1"
broadcast_address: "192.168.1.255"
ntp_server: "192.168.1.1"
site_message: "Your site PXE Boot Server"

#################### kickstart variables


storage_device: "sda"
user_name: "admin"
user_gecos: "Admin User"
encrypted_pw: [Redacted]
SSH_KEY: "STUB"

#################### variables for mounting Rocky Linux ISO

linux_iso: http://mirrors.rit.edu/rocky/8/isos/x86_64/Rocky-8.6-x86_64-dvd1.iso
iso_target: /tmp/Rocky-8.6-x86_64-dvd1.iso
mount_dir: /mnt

```
This gets you a PXE boot server that can load interactive or basic automated installations of both Rocky Linux 8.6 or Debian 11.5. 

## Defining Servers in your environment

The base defaults file comes with two server definitions. You will only need to know the IP, MAC, NIC name to complete this section. You are also required to supply if the server will be a Rocky Linux or Proxmox server.

This example is from the default setup:

```yaml
servers:
  hostname: 
    - testpve.home.lan:
      - Server_IP: 192.168.1.21
      - Server_MAC: 40-a8-f0-40-43-86
      - Server_NIC: eno1
      - Server_TYPE: proxmox
      - Server_boot: sda
    - testrl.home.lan:
      - Server_IP: 192.168.1.11
      - Server_MAC: f8-b1-56-a8-20-b4
      - Server_NIC: eno1
      - Server_TYPE: rocky
      - Server_boot: sda
```

Dependencies
------------

This playbook assumes the server you will modify is at "192.168.1.2". In addition to modifying the variables above, **change the entry in the hosts file in the top level directory of this playbook.** You should, of course, have complete ssh access with additional ability to become root non-interactively as this is necessary for the plays.

This should work on Rocky, Alma or any other Redhat flavor. 

## Sources

The preseed file used that turns a debian install into a proxmox server in this playbook I got with very minmal changes from here:

https://forum.proxmox.com/threads/pve-7-unattended-install.93658/

All the credit in the world goes to [dakobg](https://forum.proxmox.com/members/dakobg.125981/).

License
-------

BSD
