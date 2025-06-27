- Download the targeted DGX OS ISO image from Nvidia:  
  https://enterprise-support.nvidia.com/s/downloads
- Setup a simple VM with following sample configuration:  
  Processor: 4 x vCPUs  
  Memory: 2GB  
  Single Root VD: 50GB  
  Networt: 1 x vNIC (NAT or Brigded, DHCP enabled with internet access)
  EFI Boot: Enabled (Important. DGX OS is configured for booting with EFI)  
- Kick start DGX OS image installation by selecting "Install DGX OS 7.0.2" from the GRUB boot menu:
  ![](./Images/dgx/dgx-grub-menu.png)
- Post installation, permit direct ssh as root for ease of operation instead of having to sudo everytime a elevated privilegde is required.
  ```
  login as: uadmin
  uadmin@192.168.1.6's password:
  Welcome to NVIDIA DGX on VirtualBox Version 7.0.2 (GNU/Linux 6.8.0-55-generic x86_64)

   System information as of Fri Jun 27 16:11:17 +08 2025

    System load:  0.0                Processes:               118
    Usage of /:   10.1% of 48.42GB   Users logged in:         1
    Memory usage: 12%                IPv4 address for enp0s3: 192.168.1.6
    Swap usage:   0%
  Last login: Fri Jun 27 16:09:17 2025 from 192.168.1.33
  To run a command as administrator (user "root"), use "sudo <command>".
  See "man sudo_root" for details.
  
  uadmin@dgx01:~$ sudo su -
  root@dgx01:~# echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
  root@dgx01:~# passwd
  New password:
  Retype new password:
  passwd: password updated successfully
  root@dgx01:~# systemctl restart ssh
  ```
- DGX OS image does not include the DOCA OFED software which is a pre-requisite for EXA client software installation. To add DOCA repository, create a new file named /etc/apt/sources.list.d/doca.sources with following contents:
  ```
  Types: deb
  URIs: https://linux.mellanox.com/public/repo/doca/2.9.1-2/ubuntu24.04/x86_64/
  Suites: /
  Signed-By: /usr/share/keyrings/GPG-KEY-Mellanox.gpg
  ```
  And update the newly added DOCA repository:
  ```
  # mv /etc/apt/sources.list.d/cuda-compute-repo.sources{,.disabled}
  # apt-get update
  ```
- Install DOCA OFED:
  ```
  # dpkg --purge ibsim-utils ibutils libumad2sim0
  # apt-get -o Dpkg::Options::=--force-confnew install -y doca-ofed
  # ofed_info -s
  OFED-internal-24.10-1.1.4.0.200:
  ```
- Restore the CUDA Repository which has been disabled above:
  ```
  # cd /etc/apt/sources.list.d
  # mv cuda-compute-repo.sources.disabled uda-compute-repo.sources
  ```
- Reboot VM before proceeding with EXA client installation.
- Upload the latest EXA client software package to the DGX image:
  ```
  # scp exa-client-6.3.3.tar.gz root@192.168.1.6:/root
  ```
- Objective here is to download the list of software packages dependecy for offline EXA client installation. Extract and run exa_client_deploy.py without internet connection. Prior disabling internet connection, perform one time "apt-get update" first:
 ```
  # mv /etc/apt/sources.list.d/dgx.sources /etc/apt/sources.list.d/dgx.sources.disabled
  # apt-get update
  # route del default gw 192.168.1.1
  # ping www.google.com
  ping: connect: Network is unreachable
  # cd /root
  # tar xf exa-client-6.3.3.tar.gz
  # cd exa-client/
  # ./exa_client_deploy.py

  DDN EXAScaler client software installation tool: Version 6.3.3
  Select an option:

  1) Check if DDN EXAScaler client software is installed
  2) Install DDN EXAScaler client software
  3) Configure DDN EXAScaler client software
  4) Remove DDN EXAScaler client software
  5) List DDN EXAScaler mount commands
  6) Exit
  2

  Selected /root/exa-client/lustre-source.tar.gz for installation

Preparing build environment...
Unable to prepare build environment. Failed command: DEBIAN_FRONTEND=noninteractive apt-get -y install attr autoconf automake bc bison build-essential bzip2 cpio debhelper devscripts ed fakeroot fio flex gcc gettext git golang kernel-wedge keyutils kmod krb5-multidev libaio-dev libattr1-dev libaudit-dev libbison-dev libblkid-dev libc6-dev libelf-dev libgssapi-krb5-2 libjson-c-dev libkeyutils-dev libkeyutils1 libkrb5-3 libkrb5-dev liblzma-dev libmount-dev libnl-3-dev libnl-genl-3-dev libpam0g-dev libpython3-dev libreadline-dev libselinux-dev libsnmp-dev libssl-dev libtool libtool-bin libudev-dev libyaml-dev lsof m4 make module-assistant pkg-config python3 python3-dev python3-netaddr python3-netifaces python3-setuptools quilt rsync sg3-utils swig systemd wget zlib1g-dev
```
