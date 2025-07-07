1) Setup a VM with minimal Ubuntu install and with "apt-mirror" package installed. Targeted Ubuntu release here is version 24.04, codename "Noble".  
   ```
   # apt install apt-mirror
   ```

2) Setup a target directory for mirror repository download.  The target directory must be owned by the user ``apt-mirror``.``sdb1`` device used in following steps is a USB portable storage device:
   ```
   # mkdir /mnt/repo
   # mount /dev/sdb1 /mnt/repo
   # mkdir /mnt/repo/noble-24.04-repo
   # chown apt-mirror:apt-mirror /mnt/repo/noble-24.04-repo
   ```

3) Edit ``apt-mirror`` configuration file to point to above target directory and to configure the list of repositories to download. To minimize the download size, includes only the main repositories. 32-bit (deb-i386) and security related packages can be excluded.
   ```
   # cat /etc/apt/mirror.list
   ############# config ##################
   #
   set base_path /mnt/repo/noble-24.04-repo
   #
   # set mirror_path $base_path/mirror
   # set skel_path $base_path/skel
   # set var_path $base_path/var
   # set cleanscript $var_path/clean.sh
   # set defaultarch <running host architecture>
   # set postmirror_script $var_path/postmirror.sh
   set run_postmirror 0
   set nthreads 20
   set _tilde 0
   #
   ############# end config ##############
   # Standard Canonical package repositories:
   #deb http://security.ubuntu.com/ubuntu noble-security main multiverse universe restricted
   deb http://archive.ubuntu.com/ubuntu/ noble main multiverse universe restricted
   deb http://archive.ubuntu.com/ubuntu/ noble-updates main multiverse universe restricted
   #
   #deb-i386 http://security.ubuntu.com/ubuntu noble-security main multiverse universe restricted
   #deb-i386 http://archive.ubuntu.com/ubuntu/ noble main multiverse universe restricted
   #deb-i386 http://archive.ubuntu.com/ubuntu/ noble-updates main multiverse universe restricted
   # Clean unused items
   clean http://archive.ubuntu.com/ubuntu
   clean http://security.ubuntu.com/ubuntu
   ```

4) Run ``apt-mirror`` and wait for it to finish downloading content. If download interrupted for whatever reason, simply re-run ``apt-mirror`` to resume.
   ```
   # apt-mirror
   # 
   ### Wait for hours to complete ...
   #
   # df -H /mnt/repo/noble-24.04-repo
   Filesystem      Size  Used Avail Use% Mounted on
   /dev/sdb1       503G  383G   95G  81% /mnt/repo
   ```

5) Copy/transport over above downloaded data to the target air-gapped system. For consistency, we will mount using the same mount path name:
   ```
   # mkdir /mnt/repo
   # mount /dev/sdb1 /mnt/repo
   ``` 

6) Remove all default repositories configured:
   ```
   # cat /etc/apt/sources.list
   # Ubuntu sources have moved to /etc/apt/sources.list.d/ubuntu.sources
   # mv /etc/apt/sources.list.d /etc/apt/sources.list.d.bak
   ```

7) Upload the latest EXA client software package to air-gapped system:  
   ```
   # scp exa-client-6.3.4.tar.gz root@192.168.56.123:/root
   ```

8) Run EXA client installation script to obtain the list of needed (missing) packages to be installed first:
   ```
   # cd /root
   # tar xf exa-client-6.3.4.tar.gz
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

9) Configure ``apt`` to use above mounted directory as local repository source:
   ```
   # cat /etc/apt/sources.list
   # Ubuntu sources have moved to /etc/apt/sources.list.d/ubuntu.sources
   deb file:///mnt/repo/noble-24.04-repo/mirror/archive.ubuntu.com/ubuntu/ noble main multiverse universe restricted
   deb file:///mnt/repo/noble-24.04-repo/mirror/archive.ubuntu.com/ubuntu/ noble-updates main multiverse universe restricted
   ```

10) Based on the required missing packages listed in step (8), manually install them:
   ```
   # export DEBIAN_FRONTEND=noninteractive
   # apt-get -y install attr autoconf automake bc bison build-essential bzip2 cpio debhelper devscripts ed fakeroot fio flex gcc gettext git golang kernel-wedge keyutils kmod krb5-multidev libaio-dev libattr1-dev libaudit-dev libbison-dev libblkid-dev libc6-dev libelf-dev libgssapi-krb5-2 libjson-c-dev libkeyutils-dev libkeyutils1 libkrb5-3 libkrb5-dev liblzma-dev libmount-dev libnl-3-dev libnl-genl-3-dev libpam0g-dev libpython3-dev libreadline-dev libselinux-dev libsnmp-dev libssl-dev libtool libtool-bin libudev-dev libyaml-dev lsof m4 make module-assistant pkg-config python3 python3-dev python3-netaddr python3-netifaces python3-setuptools quilt rsync sg3-utils swig systemd wget zlib1g-dev
   ```

11) After installing above packages, check again if there are further missing packages by disabling configured local repositories and re-run EXA client installation script:
   ```
   # vi /etc/apt/sources.list
   # cat /etc/apt/sources.list
   # Ubuntu sources have moved to /etc/apt/sources.list.d/ubuntu.sources
   #deb file:///mnt/repo/noble-24.04-repo/mirror/archive.ubuntu.com/ubuntu/ noble main multiverse universe restricted
   #deb file:///mnt/repo/noble-24.04-repo/mirror/archive.ubuntu.com/ubuntu/ noble-updates main multiverse universe restricted
   # apt clean
   # apt update
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
   ```

12) Repeat step (10) and (11) until no more packages reported. If reported ``linux-headers-generic`` needs to be installed as shown below:
    ```
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
    Unable to prepare build environment. Failed command: DEBIAN_FRONTEND=noninteractive apt-get -y install linux-headers-generic
    ```
   Header package should already been installed with the system. To verify, ``apt list --installed | grep linux-headers-generic``. Make sure all repositories have been removed from ``/etc/apt/sources.list`` and re-run EXA client setup script.

13) After manually installing the required missing packages, EXA client installation script should go through successfully. Above "manual" steps to assist installing the missing packages are to "control" the script on the extra packages to be installed. The imported local repositories usually will have higher kernel version. If kernel version gets upgraded automatically by the script (like with ``linux-headers-generic`` package), two issues will arise:  
    - DKMS compiled OFED packages will get recompile, extending the installation process drastically.  
    - EXA client installation script will compile against current running kernel version which is wrong. The upgraded kernel version will take effect during next reboot.  
   ```
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
    Preparing build environment... Done

    Building EXAScaler client software. This may take a while...
    EXAScaler client software packages are installed and placed in /opt/ddn/exascaler/debs folder
    Use option 3 to configure EXAScaler client software before loading lustre module

    DDN EXAScaler client software installation tool: Version 6.3.3
    Select an option:

    1) Check if DDN EXAScaler client software is installed
    2) Install DDN EXAScaler client software
    3) Configure DDN EXAScaler client software
    4) Remove DDN EXAScaler client software
    5) List DDN EXAScaler mount commands
    6) Exit
    1
    Found installed EXAScaler client software packages version 2.14.0-ddn214-1:

    lustre-client-modules-6.8.0-55-generic
    lustre-client-utils
    lustre-dev
    lipe-lpcc

    DDN EXAScaler client software installation tool: Version 6.3.3
    Select an option:

    1) Check if DDN EXAScaler client software is installed
    2) Install DDN EXAScaler client software
    3) Configure DDN EXAScaler client software
    4) Remove DDN EXAScaler client software
    5) List DDN EXAScaler mount commands
    6) Exit
    6
    ```
