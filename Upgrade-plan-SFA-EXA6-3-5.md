- [x] 1. Pre-upgrade action item (from xxx-nfsgw01).
  ```
  cd /scratch/maintenance-logs/2025-12-end-yr-maintain/emf-6.3.5-2025111300
  ./emf config automod migrate --in-place --config-file ./exascaler.toml
  export EXA_CONFIG_PATH=/scratch/maintenance-logs/2025-12-end-yr-maintain/emf-6.3.5-2025111300/exascaler.toml
  ./emf config get 'host.*.backup.paths'           
  ./emf config delete 'host.*.backup.paths' --in-place
  ./emf config set --in-place --append 'host.*.backup.paths' --string "/etc/rc.d/rc.local"
  ./emf config get 'host.*.backup.paths'           
  ```
- [x] 2. Collect pre-upgrade diags output from all controllers and lustre quota listings output (from xxx-nfsgw01).
  ```
  cd /scratch/maintenance-logs/2025-12-end-yr-maintain/20251204-preupgrade-diags-output
  sh ../commands/collect-diags.sh ../commands/c0-list
  sh ../commands/collect-diags.sh ../commands/c1-list
  ../commands/diag_mce_error_checker_v1.2.sh ./*.tgz

  lfs quota -a -u -h /lustre/scratch >> /scratch/maintenance-logs/2025-12-end-yr-maintain/20251204-preupgrade-logs/scratch-quota-usage-listing.out
  lfs quota -a -g -h /lustre/scratch >> /scratch/maintenance-logs/2025-12-end-yr-maintain/20251204-preupgrade-logs/scratch-quota-usage-listing.out
  lfs quota -a -p -h /lustre/scratch >> /scratch/maintenance-logs/2025-12-end-yr-maintain/20251204-preupgrade-logs/scratch-quota-usage-listing.out
  lfs quota -a -u -h /lustre/home >> /scratch/maintenance-logs/2025-12-end-yr-maintain/20251204-preupgrade-logs/home-quota-usage-listing.out
  lfs quota -a -g -h /lustre/home >> /scratch/maintenance-logs/2025-12-end-yr-maintain/20251204-preupgrade-logs/home-quota-usage-listing.out
  lfs quota -a -p -h /lustre/home >> /scratch/maintenance-logs/2025-12-end-yr-maintain/20251204-preupgrade-logs/home-quota-usage-listing.out
  ```
- [x] 3. Collect pre-upgrade support bundle (from xxx-nfsgw01).
  ```
  cd /scratch/maintenance-logs/2025-12-end-yr-maintain/emf-6.3.5-2025111300
  ./emf sos --sr preupgrade --all --config-file ./exascaler.toml --password
  ```
- [x] 4. Shutdown down EXA cluster (from xxx-mds01)(10mins)
  ```
  ### Disbale DDNi monitoring by setting to Maintenance Mode
  emf sfa vm set-pending-auto-startup-all --auto-start false
  emf sfa vm list
  emf ha list
  emf ha stop
  clush -a -x xxx-mds01 init 0
  init 0
  emf sfa vm list
  ```
- [x] 5. Reboot all controllers and connected enclosures (open multiexec sessions to all c0)(30mins).
  ```
  SET SUBSYSTEM OFFLINE
  SHUTDOWN ENCLOSURE 2 RESTART
  SHOW ENC (wait for ESM to report back before restarting next enclosure)
  SHUTDOWN ENCLOSURE 3 RESTART
  SHUTDOWN SUBSYSTEM RESTART
  ```
- [x] 6. Start back EXA cluster and check status (10mins).
  ```
  emf sfa vm start-all
  emf sfa vm set-pending-auto-startup-all --auto-start true
  emf ha start
  emfperf pingall
  clush -ab lustre_recovery_status
  ```
- [x] 7. Login back to xxx-nfsgw01 (base of operations) and backup files.
  ```
  cd /scratch/maintenance-logs/2025-12-end-yr-maintain/emf-6.3.5-2025111300
  export EXA_CONFIG_PATH=/scratch/maintenance-logs/2025-12-end-yr-maintain/emf-6.3.5-2025111300/exascaler.toml
  ./emf ssh pull -c ./exascaler.toml --password --backup --to /scratch/maintenance-logs/2025-12-end-yr-maintain/emf-6.3.5-2025111300/esupd-backup
  ```
- [x] 8. Destroy HA and stop EMF manager (from xxx-mds01).
  ```
  emf ha destroy
  emfctl stop
  ```
- [x] 9. Backup VM image and apply new (from xxx-nfsgw01).
  ```
  ./emf sfa image backup
  ./emf sfa image apply --from=/scratch/maintenance-logs/2025-12-end-yr-maintain/emf-6.3.5-2025111300/sw/es-6.3.5-server-rocky-2025111300-x86_64.sfa.img
  ```
- [x] 10. Set pending autostartup and boot VM from new image (from xxx-nfsgw01).
  ```
  ./emf sfa vm set-pending-auto-startup-all --auto-start=true
  ./emf sfa vm start-all
  ```
- [x] 11. Sync updated exascaler.toml to all nodes and restore files backed up earlier (from xxx-nfsgw01).
  ```
  ./emf sync ./exascaler.toml --to /etc/ddn/ --password
  ./emf ssh push -c ./exascaler.toml --mode restore --from /scratch/maintenance-logs/2025-12-end-yr-maintain/emf-6.3.5-2025111300/esupd-backup --password
  ```
- [x] 12. Login to xxx-mds01, bootstrap EMF and execute ``emf install``.
  ```
  emfctl bootstrap
  emf install --load --exclude-fs-steps --password
  ```
- [x] 13. Upgrade SFAOS (from xxx-nfsgw01).
  ```
  cp -p xxx-mds01:/root/.emf/cli.conf /root/.emf/cli.conf
  emf update run --select hca --select sfa --push /scratch/maintenance-logs/2025-12-end-yr-maintain/emf-6.3.5-2025111300/sw/es-rocky-update-pack-v6.3.5-2025111300.tar --all --password
  ```
- [x] 14. Upgrade Drive FW (from xxx-nfsgw01).
  ```
  cd /scratch/maintenance-logs/2025-12-end-yr-maintain/commands
  ./upload-firmware.sh ./stor003-012-list ../emf-6.3.5-2025111300/sw/WUH722020BL5204-DS03.DDN
  update pd * file=WUH722020BL5204-DS03.DDN online (multiExec from MobaXterm)
  show pd
  ```
- [x] 15. Reinstate rsyslog setup (from xxx-mds01).
  ```
  echo "*.* @10.104.4.10:514    # Use @ for UDP protocol" >> /etc/rsyslog.conf
  sync-file /etc/rsyslog.conf
  clush -a systemctl restart rsyslog.service
  clush -a systemctl status rsyslog.service
  ```
- [x] 16. Post check.
  ```
  emf sfa health
  emfperf pingall
  emf ha list
  clush -ab lustre_recovery_status
  emf sfa ioc list (IB current:20.43.2566  new:20.43.3608)
  emf sfa vm list
  emf sfa controller list (current:12.7.0  new:12.8.0)
  modinfo lustre | grep version (current:2.14.0_ddn205  new:2.14.0-ddn323)
  mount_lustre_client
  lfs check mdts; lfs check osts; lfs df -h; df -h
  clush -ab "lctl get_param osd-*.*.quota_slave.enabled"
  clush -ab "lctl get_param mdt.*.identity_upcall"
  clush -ab chronyc sources
  ```
- [x] 17. es-log-agent setup:
  ```
  [root@xxx-mds01 ~]# es-log-cli --configure
  Below information is required to configure es-log-agent

  Note: Fields marked with * are mandatory.

  1. Customer Name* []: xxx
  2. Company Name* []: xxx
  3. Cluster/FileSystem Name* []: xxx-es400nvx2
  4. Primary Server IP address* (for failover) [10.104.2.113]:
  5. Contact Email (Optional) []:
  6. Site / Location (Optional) []:
  7. DDN recommends to enable nightly bundle. Do you want to enable it? (Y/n) [Y]: n
  Nightly bundles will not be setup.

  ==================================================
  CONFIGURATION SUMMARY
  ==================================================
  1. Customer Name: xxx
  2. Company Name: xxx
  3. Cluster Name or ID: xxx-es400nvx2
  4. Primary Server IP (for failover): 10.104.2.113
  5. Contact Email:
  6. Site / Location:
  7. Nightly bundles with SIA: Disabled
  ==================================================
  Saving configuration to /etc/ddn/es-log-agent/agentconfig.toml
  Checking connection to other hosts
  Able to ping all the vms
  Connections to others host successful
  Agent configured properly.
  ```
  ```
  [root@xxx-mds02 ~]# es-log-cli  --configure
  Below information is required to configure es-log-agent

  Note: Fields marked with * are mandatory.

  1. Customer Name* []: xxx
  2. Company Name* []: xxx
  3. Cluster/FileSystem Name* []: xxx-es400nvx2
  4. Primary Server IP address* (for failover) [10.104.2.113]:
  5. Contact Email (Optional) []:
  6. Site / Location (Optional) []:
  7. DDN recommends to enable nightly bundle. Do you want to enable it? (Y/n) [Y]: n
  Nightly bundles will not be setup.

  ==================================================
  CONFIGURATION SUMMARY
  ==================================================
  1. Customer Name: xxx
  2. Company Name: xxx
  3. Cluster Name or ID: xxx-es400nvx2
  4. Primary Server IP (for failover): 10.104.2.113
  5. Contact Email:
  6. Site / Location:
  7. Nightly bundles with SIA: Disabled
  ==================================================
  Saving configuration to /etc/ddn/es-log-agent/agentconfig.toml
  Checking connection to other hosts
  Able to ping all the vms
  Connections to others host successful
  Agent configured properly.
  ```
