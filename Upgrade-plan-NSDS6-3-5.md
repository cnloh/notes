- [X] 1. Pre-upgrade check.
  ```
  # lsb_release -a
  LSB Version:    :core-4.1-amd64:core-4.1-noarch:cxx-4.1-amd64:cxx-4.1-noarch
  Distributor ID: Rocky
  Description:    Rocky Linux release 8.10 (Green Obsidian)
  Release:        8.10
  Codename:       GreenObsidian

  # modinfo lustre | grep "^version"
  version:        2.14.0_ddn205

  # ofed_info -s
  OFED-internal-24.10-2.1.8:

  # nsds -vv
  nsdataservice  : 6.3.3-27fc412
  samba          : 4.20.7-ddn.9e61ad7
  nfs-ganesha    : 5.9-ddn.e51f847
  sssd           : 2.9.5
  container OS   : Rocky Linux 8.10 (Green Obsidian)

  # emf --version
  emf 6.3.3-2025052100
  ```
- [x] 2. Pre-upgrade logs collection.
  ```
  emf sos --local
  nsds diag collect
  ```
- [x] 3. Stop NSDS cluster (from xxx-nfsgw01).
  ```
  nsds cluster stop
  ```
- [x] 4. Create a backup for NSDS (from xxx-nfsgw01).
  ```
  /scratch/maintenance-logs/2025-12-end-yr-maintain/nsds-upgrade/nsds_backup.sh (do save the output instructions for restoration)
  ```
- [x] 5. Umount and unload lustre module.
  ```
  umount /lustre/home
  lustre_rmmod
  ```
- [x] 6. Mount the EXAScaler ISO to be upgraded to and manually install emf, client and server repo rpms from the ISO image (both servers).
  ```
  mount /scratch/es-6.3.5-server-rocky-2025111300-x86_64.iso /mnt/iso
  cd /mnt/iso/BaseOS/Packages/emf
  dnf --disablerepo=* localinstall emf-repo-el8-2025111300-0.x86_64.rpm -y

  cd /mnt/iso/BaseOS/Packages/lustre
  dnf --disablerepo=* localinstall lustre-client-repo-6.3.5-ddn232.x86_64.rpm -y
  dnf --disablerepo=* localinstall lustre-server-repo-rocky8.10-6.3.5-ddn232.x86_64.rpm -y
  ```
- [x] 7. Edit or create yum repositories configuration file to reflect new repo available from the ISO image (both servers).
  ```
  cat /etc/yum.repos.d/ddn-es.repo
  [ddn-es]
  name=DDN EXA Base
  baseurl=file:///mnt/iso/BaseOS/
  module_hotfixes=1
  enabled=1
  gpgcheck=0

  [ddn-es-updates]
  name=DDN EXA Updates
  baseurl=file:///mnt/iso/Updates/
  module_hotfixes=1
  enabled=0
  gpgcheck=0

  [emf]
  name=EMF Repo
  baseurl=file:///scratch/repo/emf/el8
  module_hotfixes=1
  enabled=1
  gpgcheck=0

  [es-server]
  name=ES Server Repo
  baseurl=file:///scratch/repo/lustre/server/rocky8.10
  module_hotfixes=1
  enabled=1
  gpgcheck=0
  ```
- [x] 8. Initiate full upgrade from the newly configured repositories (both servers; each server takes around 15mins to complete).
  ```
  dnf repolist
  dnf clean all
  dnf remove fence-agents-vbox-4.2.1-112.el8.noarch (this package will cause upgrade check error. Just remove it)
  dnf --disablerepo=* --enablerepo=ddn-es,emf,es-server distro-sync
  vi /etc/fstab (to prevent lustre from auto-mounting first)
  reboot
  ```
- [x] 9. Post-upgrade versions check.
  ```
  lsb_release -a
  ofed_info -s
  modinfo lustre | grep "^version"
  nsds -vv
  emf --version
  vi /etc/fstab (to enable back lustre auto-mounting)
  mount /lustre/home
  ```
- [x] 10. NSDS configuration upgrade (from xxx-nfsgw01).
  ```
  systemctl status chrony-wait
  nsds config upgrade apply
  ```
  Actual output:
  ```
  [root@xxx-nfsgw01 ~]# nsds config upgrade apply
  [ INFO  ] [2025-12-04T18:14:49] SMB service is disabled, relevant prereq  checks would be skipped.
  [ INFO  ] [2025-12-04T18:14:49] Running the prerequisites checks on nodes 10.104.1.28, 10.104.1.27.

  Collecting prereq facts...
  Done.

  Saved prereq check run report: /var/log/nsdataservice/prereq_reports/ddn-nsds-prereq-report-start-services-2025-12-04_18_14_52.984354+08_00.json
  [ INFO  ] [2025-12-04T18:14:52] All required prerequisites checks passed.

  Checking upgrade eligibility of the node(s)..
  +----------------------------------------+--------------------+-------+
  | Node                                   | Result             | Error |
  +----------------------------------------+--------------------+-------+
  | xxx-nfsgw01.xxx.xxx.xxx@10.104.1.27 | Upgrade check [OK] | -     |
  +----------------------------------------+--------------------+-------+
  | xxx-nfsgw02.xxx.xxx.xxx@10.104.1.28 | Upgrade check [OK] | -     |
  +----------------------------------------+--------------------+-------+

  Applying upgrades...
  +----------------------------------------+---------------------------------------------------+-------+
  | Node                                   | Result                                            | Error |
  +----------------------------------------+---------------------------------------------------+-------+
  | xxx-nfsgw01.xxx.xxx.xxx@10.104.1.27 | Upgraded successfully to version 'v6.3.5-8bf3105' | -     |
  +----------------------------------------+---------------------------------------------------+-------+
  | xxx-nfsgw02.xxx.xxx.xxx@10.104.1.28 | Upgraded successfully to version 'v6.3.5-8bf3105' | -     |
  +----------------------------------------+---------------------------------------------------+-------+
  ```
- [x] 11. Verify the required lustre filesystem can be auto-mounted and proceed to start back NSDS.
  ```
  df -t lustre
  nsds cluster start
  nsds cluster status
  nsds export nfs list
  ```
- Above all steps took around 2hrs to complete.
