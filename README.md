# Overview

This is a small set of Ansible roles that configure a Linux VM to be a JumpServer for a Solaris 10 host.

# Terms
* host = your local physical system
* VM = your Virtualbox Debian Linux VM
* target = your physical Sun system

# Requirements
* Mac or Linux host
* Download a Virtualbox Linux Image. 
  * I have only tested with Debian Jessie from https://www.osboxes.org/debian/ 
* Download Solaris 10 DVD iso from Oracle
* Virtualbox
* Ansible
* git

# Usage
* Start the VM
* Copy the Solaris DVD ISO to /iso/<filename.iso> in the VM
  * use rsync
* git clone this repo
* cd to this repo
* Make a new file ~/.vault_pass.txt with your vault password in it.
 * Delete group_vars/default/vault.yml , and make your own file that looks like this:

```
---
su_password: your_passwd
```

  * Then encrypt it:
```
ansible-vault encrypt group_vars/default/vault.yml --vault-password-file ~/.vault_pass.txt
```
* modify inventory.yml to suit your network
* run ansible-playbook
```ansible-playbook  -i inventory.yml -u osboxes setup_jumpserver.yml -k  --become-method=su --vault-password-file ~/.vault_pass.txt```
* When prompted, the default osboxes password is 'osboxes'
* Connect the target Sun host directly to your physical host
  * Solaris netboot will fail out if it detects incorrect DHCP or other Layer2 frames

# Notes
## files 
/etc/bootparams
```
ultra    root=linux:/solaris/Solaris_10/Tools/Boot \
         install=linux:/solaris \
         boottype=:in \
         rootopts=linux:rsize=32768
```

/etc/exports
```
/data/jumpstart    192.168.0.0/24(fsid=0,ro,no_root_squash,crossmnt,no_subtree_check,sync)
/data/jumpstart    10.0.0.0/8(fsid=0,ro,no_root_squash,crossmnt,no_subtree_check,sync)
/data/jumpstart    127.0.0.0/8(fsid=0,ro,no_root_squash,crossmnt,no_subtree_check,sync)
```

# services
/etc/init.d/bootparamd
/etc/init.d/rarpd
nfs-kernel-server
rpcbind


# tasks
```
sed -i 's/^RPCMOUNTDOPTS.*/RPCMOUNTDOPTS="--manage-gids --no-nfs-version 4"/' /etc/default/nfs-kernel-server
sed -i 's/^NEED_STATD.*/NEED_STATD=yes/' /etc/default/nfs-common
```

## on physical
```
rsync -av /Volumes/solarisdvd/ ./solaris/.
```

## on vm 
```
rsync -av /vagrant/solaris10dvd/ /data/jumpstart/solaris/.
```

exportfs *:/data/jumpstart -o fsid=0,ro,no_root_squash,crossmnt,no_subtree_check,sync
