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