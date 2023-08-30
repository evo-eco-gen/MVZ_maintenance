# Update the old MVZ Genome Assembly server to a current version of  Ubuntu Debian.
***********************************************************************************

The server was running 16.04, which was last supported in April 2021.
This posed various version incompatibility issues and security risks.
The update was a multistep procedure to avoid the risks of going directly to 22.04. The /boot directory was cleaned of o0ld kernels to create space for the install.

Update 16.04 to 18.04:
```
sudo do-release-upgrade -c
sudo apt-get update
sudo apt-get upgrade
```
Reboot. Takes a couple of minutes:
```
reboot
```
Now upgrade:
```
sudo do-release-upgrade
```
```python
Your python3 install is corrupted. Please fix the '/usr/bin/python3' symplink
Some discussion of this issue: https://askubuntu.com/questions/1104052/your-python3-install-is-corrupted
```
```
sudo ln -sf /usr/bin/python2.7 /usr/bin/python

sudo do-release-upgrade
```
```python
#(appstreamcli:5527): GLib-CRITICAL **: g_strchug: assertion 'string != NULL' failed

## Out of disk space on boot. Find what old kernels are still stored and can be removed:
```
```
du -h /boot
ls -l
```
```python
-rw-r--r-- 1 root root 43180709 Nov 12  2020 initrd.img-4.4.0-194-generic
-rw-r--r-- 1 root root 43180625 Apr 17  2021 initrd.img-4.4.0-209-generic
```
```
uname -r
uname --all
sudo apt autoremove --purge
```
These commands do not work, but it is normal for multiple commands to fail here. This clarifies the situation:
```
dpkg -l | tail -n +6 | grep -E 'linux-image-[0-9]+' | grep -Fv $(uname -r)
```
```python
rc  linux-image-4.4.0-193-generic              4.4.0-193.224                                   amd64        Signed kernel image generic
ii  linux-image-4.4.0-194-generic              4.4.0-194.226                                   amd64        Signed kernel image generic
rc  linux-image-4.4.0-197-generic              4.4.0-197.229                                   amd64        Signed kernel image generic
rc  linux-image-4.4.0-198-generic              4.4.0-198.230                                   amd64        Signed kernel image generic
 ...
ii  linux-image-4.4.0-209-generic              4.4.0-209.241                                   amd64        Signed kernel image generic
```
rc = already removed \
ii = can be removed
```
sudo dpkg --purge linux-image-4.4.0-194-generic
```
```python
dpkg: dependency problems prevent removal of linux-image-4.4.0-194-generic:
linux-modules-extra-4.4.0-194-generic depends on linux-image-4.4.0-194-generic | linux-image-unsigned-4.4.0-194-generic; however:
  Package linux-image-4.4.0-194-generic is to be removed.
 Package linux-image-unsigned-4.4.0-194-generic is not installed.

dpkg: error processing package linux-image-4.4.0-194-generic (--purge):
 dependency problems - not removing
Errors were encountered while processing:
 linux-image-4.4.0-194-generic
```
```
sudo apt purge linux-image-4.4.0-194-generic
sudo apt purge linux-image-4.4.0-209-generic
```
Warnings:
```python
W: Possible missing firmware /lib/firmware/ast_dp501_fw.bin for module ast
Purging configuration files for linux-image-4.4.0-209-generic (4.4.0-209.241) ...
rmdir: failed to remove '/lib/modules/4.4.0-209-generic': Directory not empty
```

Reboot while ignoring inhibitors related to specific users:
```
systemctl reboot -i
```
A warning about updating through ssh (risky!):

```python
To make recovery in case of failure easier, an additional sshd will 
be started on port '1022'. If anything goes wrong with the running 
ssh you can still connect to the additional one. 
If you run a firewall, you may need to temporarily open this port. As 
this is potentially dangerous it's not done automatically. You can 
open the port with e.g.: \
'iptables -I INPUT -p tcp --dport 1022 -j ACCEPT' 
```

Other warnings along the way:
```python
52 installed packages are no longer supported by Canonical. You can 
still get support from the community. 

96 packages are going to be removed. 646 new packages are going to be 
installed. 2160 packages are going to be upgraded. 

You have to download a total of 2,512 M. This download will take 
about 3 minutes with your connection. 

Installing the upgrade can take several hours. Once the download has 
finished, the process cannot be canceled.


man-db depends on dsbmaninutils

Removing texlive-binaries (2015.20160222.37495-1ubuntu0.1) ...
dpkg: man-db: dependency problems, but processing triggers anyway as you requested:
 man-db depends on bsdmainutils; however:
  Package bsdmainutils is not configured yet.
  
  
  Errors were encountered while processing:
 /var/cache/apt/archives/docker.io_20.10.21-0ubuntu1~18.04.3_amd64.deb
Exception during pm.DoInstall():  E:Sub-process /usr/bin/dpkg returned an error code (1)
```

Verify version:
```
lsb_release -a
```
################################################################################################

Upgrade to ubuntu 20:
```
sudo do-release-upgrade -c
sudo apt-get update
sudo apt --fix-broken install
```
Problems with someone's docker:
```python
Errors were encountered while processing:
/var/cache/apt/archives/docker.io_20.10.21-0ubuntu1~18.04.3_amd64.deb
E: Sub-process /usr/bin/dpkg returned an error code (1)
```

Nuke Docker and upgrade:
```
sudo apt remove docker.io

sudo apt autoremove
sudo apt-get upgrade
sudo reboot
```
################################################################################################

## Finally - upgrade to 22.04!
```
sudo apt-get update
sudo apt-get upgrade
```
```python
#E: mkinitramfs failure cpio 141 lz4 -9 -l 24
#E: Sub-process /usr/bin/dpkg returned an error code (1)
```
```
sudo apt-get install --reinstall initramfs-tools initramfs-tools-core
sudo apt autoremove
sudo apt-get upgrade
```
Free up space on /boot once again:

```python
linux-image-4.15.0-213-generic

sudo apt purge linux-image-4.15.0-213-generic
sudo rm -rf /lib/modules/4.15.0-213-generic
sudo rm /boot/System.map-4.15.0-213-generic
sudo rm /boot/config-4.15.0-213-generic
sudo do-release-upgrade
reboot
```

################################################################################################

## Set up the Uncomplicated Firewall:

Check if IPv6 enabled:
```
sysctl -a 2>/dev/null | grep disable_ipv6
sudo vim /etc/default/ufw
```
Start with default config:
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
```
Allow incoming ssh, on ports 22 and 1022 (as a back up)
```
sudo ufw allow ssh
sudo ufw allow 1022
```
Allow ports for X11:
```
sudo ufw allow 6000:6007/tcp
```
Allow MVZ subnet: \
(TBC!)

## **ET VOILÁ!**

