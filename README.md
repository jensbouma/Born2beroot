**Born2BeRoot the Hardway**

**WORK IN PROGRESSS!!!**

This project is a guide to use terraform (and maybe Ansible) to install the complete Born2beroot subject form code without a helpertool like VirualBox or UTM to run the VM. Offcourse the created qcow2 file can run in UTM as well or (be converted to vdi to) run in VirtualBox.

**Install Homebrew**
```
  /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
**Install quemu & libvirt & cdrtools**
```
  brew install qemu libvirt cdrtools 
```

<!-- **Install virtmanager**
```
brew tap arthurk/homebrew-virt-manager
brew install virt-manager virt-viewer
``` -->

**Disable QEMU security features (as macos doesn't support this**
```
echo 'security_driver = "none"' >> /opt/homebrew/etc/libvirt/qemu.conf
echo "dynamic_ownership = 0" >> /opt/homebrew/etc/libvirt/qemu.conf
echo "remember_owner = 0" >> /opt/homebrew/etc/libvirt/qemu.conf
```

**Install vde_vmnet**
https://github.com/lima-vm/socket_vmnet
```
git clibe https://github.com/lima-vm/socket_vmnet.git
cd socket_vmnet
sudo make PREFIX=/opt/socket_vmnet install
```

**Install Libvirt service**
```
brew services start libvirt
```

**Install Hasicorp Terraform**
```
  brew install kreuzwerker/taps/m1-terraform-provider-helper
  brew tap hashicorp/tap
  brew install hashicorp/tap/terraform
  m1-terraform-provider-helper activate
  m1-terraform-provider-helper install hashicorp/template -v v2.2.0
```

**- Deploy Terraform code **
```
export TERRAFORM_LIBVIRT_TEST_DOMAIN_TYPE="qemu" 
terraform init
terraform plan
terraform apply
```

**Usefull Virsh commands**
```
virsh list
virsh start 'Debian Cloud'              // Run VM
virsh shutdown 'Debian Cloud'           // Shutdown VM
virsh reboot  'Debian Cloud'            // Reboot
virsh destroy 'Debian Cloud'            // Force Shutdown
virsh undefine --nvram 'Debian Cloud'   // Remove VM
virsh console 'Debian Cloud'            // Connect to serial console

ssh -p 2222 root@localhost  // Connect to VM with SSH
```

**Create disk of 8225M**

apt update
sudo apt-get install cryptsetup
sudo apt-get install lvm2

<!-- sudo dd if=/dev/zero of=/dev/nvme2n1 bs=512k count=16450 -->

sudo sgdisk -og /dev/nvme2n1
sudo sgdisk -og /dev/nvme2n1 && sudo sgdisk -n 1:0:+487M -n 2:0:+1K -n 3:0:+7680M  /dev/nvme2n1

dd bs=512 count=4 if=/dev/random of=/home/debian/keyfile iflag=fullblock

sudo cryptsetup -c aes-xts-plain -y -s 512 -h sha512 luksFormat --key-file=/home/debian/keyfile /dev/nvme2n1p3

<!-- sudo cryptsetup --cipher=aes-xts-plain64 --offset=0 --key-file=/home/debian/keyfile --key-size=512 open --type=plain /dev/nvme2n1p3 crypt -->

<!-- sudo cryptsetup luksFormat --key-file=/home/debian/keyfile  /dev/nvme2n1p3 -->
sudo cryptsetup luksOpen --key-file=/home/debian/keyfile /dev/nvme2n1p3 crypt

sudo pvcreate /dev/mapper/crypt

sudo vgcreate jbouma-vg /dev/mapper/crypt 
sudo lvcreate -L 2.750G -n root jbouma-vg
sudo lvcreate -L 976M -n swap_1 jbouma-vg
sudo lvcreate -L 3.750G -n home jbouma-vg


sudo mkfs.ext4 -L root /dev/jbouma-vg/root 
sudo mkfs.ext4 -L root /dev/jbouma-vg/swap_1 
sudo mkfs.ext4 -L root /dev/jbouma-vg/home


<!-- sudo dd if=/dev/nvme1n1p1 of=/dev/jbouma-vg/root bs=512 -->
<!-- resize2fs /dev/sda1 -->

sudo mkswap /dev/jbouma-vg/swap_1 
swapon /dev/jbouma-vg/swap_1 

https://discourse.brew.sh/t/failed-to-connect-socket-to-var-run-libvirt-libvirt-sock-no-such-file-or-directory/1297/2

 
<!-- 

// Need to know how to setup a account

// Must be active on startup UFW instead of default TO install probally need dnf

/* Need strong password policy */
/* Every 30 Days */
/* Min Dats Allowed defore set to 2 */
/* Pasword at least 10 chars, 1 uppercase 1 lowercase 1 number and not more than 3 consecutive indentical characters */
/* Not include the name of the user */
/* At least 7 characters that are not part of the former password (not for root)*/

/* Change all the passowrds of the users after configuration */

/* Autentication using sudo max 3 attempts */
/* Custom message if an error due wrong password occurs when using Sudo */

/* each sudo action should be archived, both input and output > /var/log/sudo/ */

/* TTY mode enabled */

/* For security reasons too, the paths that can be used by sudo must be restricted.
Example: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin */


/* Need to install and configure sudo following strict rules */
/* In addition to the rood user there sould be one with the intralogin as username */
/* This user beloings to the groups user42 and sudo groups */

// During the defense need to create user and assing it to a group

/* 
Finally, you have to create a simple script called monitoring.sh. It must be devel-
oped in bash.
At server startup, the script will display some information (listed below) on all ter- minals every 10 minutes (take a look at wall). The banner is optional. No error must be visible.
Your script must always be able to display the following information:
• The architecture of your operating system and its kernel version.
• The number of physical processors.
• The number of virtual processors.
• The current available RAM on your server and its utilization rate as a percentage. • The current available memory on your server and its utilization rate as a percentage. • The current utilization rate of your processors as a percentage.
• The date and time of the last reboot.
• Whether LVM is active or not.
• The number of active connections.
• The number of users using the server.
• The IPv4 address of your server and its MAC (Media Access Control) address. • The number of commands executed with the sudo program. */

/* 
During the defense, you will be asked to explain how this script
works.  You will also have to interrupt it without modifying it. ?????????
Take a look at cron. */

// Generate Signature -->