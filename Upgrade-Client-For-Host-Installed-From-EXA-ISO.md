- On the client host, download and install latest "lustre-client-repo-xxx.x86_64.rpm" to extract the client package. Example:
```
# dnf --disablerepo=* localinstall ./lustre-client-repo-6.3.2-ddn184.x86_64.rpm
# cd /scratch/EXAScaler-6.3.2
# tar xf exa-client-6.3.2.tar.gz
```
- Uninstall current installed lustre client:
```
# cd /scratch/EXAScaler-6.3.2/exa-client
# ./exa_client_deploy.py

DDN EXAScaler client software installation tool: Version 6.3.2
Select an option:

1) Check if DDN EXAScaler client software is installed
2) Install DDN EXAScaler client software
3) Configure DDN EXAScaler client software
4) Remove DDN EXAScaler client software
5) List DDN EXAScaler mount commands
6) Exit
4
```
- Next download and install the server repo that matches current installed EXA version. Example:
```
# cat /etc/es_install_version
EXAScaler SFA Rocky 6.3.2-2024122100

# dnf --disablerepo=* localinstall ./lustre-server-repo-rocky8.10-6.3.2-ddn184.x86_64.rpm
```
- Create the server repository configuration file and install both kernel-devel and kernel-headers package that matches the current kernel version. Client installation will look for these two packages and error out if not found or can't install:
```
# cat /etc/yum.repos.d/ddn-es.repo
[es-server]
name=ES Server Repo
baseurl=file:///scratch/repo/lustre/server/rocky8.10
module_hotfixes=1
enabled=1
gpgcheck=0

# uname -r
4.18.0-553.27.1.el8_lustre.ddn17.x86_64

# yum -y install kernel-devel-4.18.0-553.27.1.el8_lustre.ddn17.x86_64
# yum -y install kernel-headers-4.18.0-553.27.1.el8_lustre.ddn17.x86_64
```
- After both kernel packages have been installed, the server repo rpm should be uninstalled. If not later client installation will flag out client software already installed and abort:
```
# dnf remove lustre-server-repo-rocky8.10
# mv  /etc/yum.repos.d/ddn-es.repo /etc/yum.repos.d/ddn-es.repo.disabled
```
- Now can proceed to install (upgrade) lustre client as usual using "exa_client_deploy.py" script. Prior do ensure following repo have been enabled:
```
# cat /etc/yum.repos.d/Rocky-BaseOS.repo
# cat /etc/yum.repos.d/Rocky-AppStream.repo
# cat /etc/yum.repos.d/Rocky-Devel.repo
# /scratch/EXAScaler-6.3.2/exa-client/exa_client_deploy.py
```
- If there is a need to uninstall MOFED:
```
# /usr/sbin/ofed_uninstall.sh
```
