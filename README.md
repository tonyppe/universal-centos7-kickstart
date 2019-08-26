# Universal KS

KS script to boot computers and install CentOS 7 with goals: 

  - Automated and unattended as much as possible
  - root and admin users and passwords set
  - disk encryption
  - auto detection of HDD, create / partition to 100% of the disk size regardless of size

How to use

  - Boot the computer from standard CentOS 7 install USB media
  - At the menu, edit the command and add the location of the kickstart script on the network
  - System must have network plugged in to be able to reach the kickstart
  
## Script Instructions
  
  1. Download and save the script to a web server that your system can reach over the network
  2. Modify the script to suit your needs (See modifications below)
  3. When ready, boot the system using the usb installation drive (See CentOS web site for details how to create the usb boot drive)
  4. On the system before booting - edit the grub command line and replace `quiet` with the location of the kickstart. eg. ks=https://raw.githubusercontent.com/tonyppe/universal-centos7-kickstart/master/uni-ks.cfg
  5. Watch the system auto-install
  
## Modifications

Most of the script contains comments to help you. Some important ones listed below

### System language and timezone
Currently set to Australia:
 `lang en_AU.UTF-8`
#### System timezone
timezone Australia/Perth --isUtc

### Root password 
Create your own `root` user password by using `openssl` and run `openssl passwd -1 "your-password-here"`
`rootpw --iscrypted $1$9ZV7FJKp$XeX615v1GFG9FFZQujNKD0`

### Disk partitioning with encryption
Encryption passphrase will prompt on install else add '--passphrase=something' after --encrypted line. Example below.
No swap partition because this uses a swapfileinstead which can be more easily increased / decreased / moved after OS install.
'part /boot/efi --fstype="efi" --asprimary --size=200 --fsoptions="umask=0077,shortname=winnt"'
'part /boot --fstype="xfs" --asprimary --size=2048'
'part pv.226 --fstype="lvmpv" --asprimary --grow --size=51205 --encrypted --passphrase=something'
'volgroup centos --pesize=4096 pv.226'
'logvol /  --fstype="xfs" --grow --size=51200 --label="/" --name=root --vgname=centos'

## Post install section

1. Need to add DNS server so the system can resolve DNS at the post install section
`echo "nameserver 1.1.1.1" >> /etc/resolv.conf`

2. Update the sudoers file to allow users in the wheel group to run sudo without entering a password
`sudo sed -i 's/^#\s*\(%wheel\s\+ALL=(ALL)\s\+NOPASSWD:\s\+ALL\)/\1/' /etc/sudoers
sudo sed -i 's/^\(%wheel\s\+ALL=(ALL)\s\+ALL\)/#\1/' /etc/sudoers`

3. update grub timeout to `0` to hide grub menu
`sed -i 's/GRUB_TIMEOUT=5/GRUB_TIMEOUT=0/g' /etc/default/grub
grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg`

4. Don't allow user `root` to log in via ssh since ssh access is open
`echo "PermitRootLogin no" >> /etc/ssh/sshd_config`

5. If you want to disable the `root` account completely (meaning not even `sudo su` is available) then uncomment the following from the script
`# disable root account (cant sudo su)
# sed -i '/^root/ s/\/bin\/bash/\/sbin\/nologin/' /etc/passwd`

6. Configure an 8GB swapfile in the / area using the below. Change `count=8000` to another value if you want a different size swap. For example for (about) 16GB swap use `count=16000`
`# add a 8GB swapfile
dd if=/dev/zero of=/swapfile bs=1M count=8000
chmod 600 /swapfile
mkswap /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab`

7. If you need an internal CA certificate to be imported, so the system will trust sites and services with certificates published by the CA then add the CA PEM and uncomment the lines
`# import internal CA cert placeholder
#touch /etc/pki/ca-trust/source/anchors/juc-noc1-prd-ca.crt
#echo "-----BEGIN CERTIFICATE----- 
#-----END CERTIFICATE-----
#" > /etc/pki/ca-trust/source/anchors/juc-noc1-prd-ca.crt
#update-ca-trust`

8. Install epel repo and XFCE
`yum install epel-release -y
yum groupinstall xfce -y`

9. Remove firefox browser because it's too old to be useful in this set up. Note: Chrome browser is installed using a post-boot.sh script. If you dont have a post-boot.sh then you may wish to keep firefox so that the system has a browser
`yum remove firefox -y`

10. install the following to resolve gnome bug with Cisco Anyconnect SSL VPN
`yum install NetworkManager-openconnect-gnome -y`

11. Download and save a post-boot.sh if you want it
`#wget -O /opt/post-boot.sh http://1.1.1.1/tony/post-boot.sh 
#chmod +x /opt/post-boot.sh `

12. Save custom web shortcuts on the desktop which will be placed on the desktop of any user logging in
`mkdir /etc/skel/Desktop`

`touch /etc/skel/Desktop/WebexTeams.desktop
echo "[Desktop Entry]
Encoding=UTF-8
Name=Webex Teams
Type=Link
URL=https://teams.webex.com/
Icon=text-html" > /etc/skel/Desktop/WebexTeams.desktop`

Additional shortcuts can be created by duplicating and modifying the above



