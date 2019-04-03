# TL; DR

* we guide is step-by-step guide for PXE boot env with UBNT/Synology.
  * You can setup with your own TFTPd with docker or packages.

# Basic Steps
## Step 1 - ipxe build
* build with docker.
  * we will use ubuntu docker image.

```
apt update && apt install -fy vim wget git unzip build-essential liblzma-dev syslinux mkisofs
wget https://github.com/ipxe/ipxe/archive/master.zip
unzip master.zip
cd ipxe-master/src

# Edit config/general.h to setup. or give proper defines.
# > this will be support of HTTPS or some...
vi config/general.h

# Edit Injected file
vi boot.ipxe


make bin/undionly.kpxe EMBED=boot.ipxe
```

* example boot.ipxe will just like this
  * You can specify chaining of some pxe configuration path/URL.

```
root@57477e3ee3dc:~/ipxe-master/src# cat boot.ipxe
#!ipxe

dhcp
chain http://10.1.2.50/pxe/boot
```


* You will get bin/undionly.kpxe and copy it into proper tftp server file path.

## Step 2 - prepare Synology
* Add setting for TFTP.
  * copy compiled ipxe (undionly.kpxe) into TFTP path. (we use it as /pxeboot/)
  * rename it as /pxeboot/pxelinux.0
* Add Web Server to store pxe configuration.

## Step 3 - prepare UBNT dhcpd.

* You must set up DHCPd.
* You must login with SSH.


```
# logged as user (not root)
configure

#Add bootfile-server option to DHCP config
# bootfile-server must be Synology(TFTP) server host ip.
edit service dhcp-server shared-network-name SOME_DHCP_NAME subnet 10.1.2.0/24
set bootfile-server 10.1.2.50

#Add filename option to DHCP config. Make sure to encapsulate the file path in &quot;
set subnet-parameters "filename &quot;/pxeboot/pxelinux.0&quot;;"

#Depending on your configuration, you may also need to add the bootfile-name parameter
set bootfile-name /pxeboot/pxelinux.0

#Commit and save
commit
save
```


## Step 4 - prepare pxe configures

# OS Specific
## Ubuntu Binary

* get files from ubuntu mirror to boot
  * you can find current installer kernel and binary from mirror's path,
    * "/ubuntu/dists/bionic/main/installer-amd64/current/images/netboot/ubuntu-installer/amd64/"
    * file "initrd.gz" and "linux"
