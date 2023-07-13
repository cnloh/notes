- From the VM hosting EMF services, install the repo hotfix packages:
```
# dnf install emf-repo-rpm-6.2.0-2.el8.x86_64.rpm
# dnf install emf-repo-deb-6.2.0-2.el8.x86_64.rpm
```
- Enable EMF repo:
```
# vi /etc/yum.repos.d/emf.repo
# grep enabled /etc/yum.repos.d/emf.repo
enabled=1
```
- Verify EMF repo has been enabled:
```
# dnf repolist all
repo id                                repo name                               status
emf                                    emf repo                                enabled
```
- Clear repo cache and begin updates from the updated EMF repo:
```
# dnf clean all
8 files removed
# dnf update
emf repo                                                                                              948 kB/s |  82 kB     00:00
Dependencies resolved.
======================================================================================================================================
 Package                                          Architecture             Version                        Repository             Size
======================================================================================================================================
Upgrading:
 emf-action-agent                                 x86_64                   6.2.0-2.el8                    emf                   2.7 M
 emf-agent                                        x86_64                   6.2.0-2.el8                    emf                   1.9 M
 emf-agent-target                                 x86_64                   6.2.0-2.el8                    emf                   8.7 k
 emf-api                                          x86_64                   6.2.0-2.el8                    emf                   7.6 M
 emf-bootstrap                                    x86_64                   6.2.0-2.el8                    emf                   8.0 k
 emf-cli                                          x86_64                   6.2.0-2.el8                    emf                   6.8 M
 emf-cli-bash-completion                          x86_64                   6.2.0-2.el8                    emf                    12 k
 emf-cluster-cli                                  x86_64                   6.2.0-2.el8                    emf                   3.2 M
 emf-config-cli                                   x86_64                   6.2.0-2.el8                    emf                   5.3 M
 emf-config-cli-bash-completion                   x86_64                   6.2.0-2.el8                    emf                   8.7 k
 emf-corosync                                     x86_64                   6.2.0-2.el8                    emf                   2.2 M
 emf-corosync-agent                               x86_64                   6.2.0-2.el8                    emf                   2.1 M
 emf-device                                       x86_64                   6.2.0-2.el8                    emf                   2.5 M
 emf-device-agent                                 x86_64                   6.2.0-2.el8                    emf                   2.0 M
 emf-device-scanner                               x86_64                   6.2.0-10.el8                   emf                   2.3 M
 emf-embedded                                     x86_64                   6.2.0-2.el8                    emf                   7.9 k
 emf-es-fence-agents                              x86_64                   6.2.0-2.el8                    emf                   1.7 M
 emf-es-tools                                     x86_64                   6.2.0-2.el8                    emf                    41 M
 emf-exa-node-config                              x86_64                   6.2.0-2.el8                    emf                   7.1 k
 emf-exa-templates                                x86_64                   6.2.0-2.el8                    emf                    40 k
 emf-gpu                                          x86_64                   6.2.0-2.el8                    emf                   2.8 M
 emf-grafana                                      x86_64                   6.2.0-2.el8                    emf                    13 k
 emf-gui                                          x86_64                   6.2.0-10.el8                   emf                   1.7 M
 emf-host                                         x86_64                   6.2.0-2.el8                    emf                   2.1 M
 emf-host-agent                                   x86_64                   6.2.0-2.el8                    emf                   1.9 M
 emf-influx                                       x86_64                   6.2.0-2.el8                    emf                   7.6 k
 emf-journal                                      x86_64                   6.2.0-2.el8                    emf                   2.2 M
 emf-journal-agent                                x86_64                   6.2.0-2.el8                    emf                   2.1 M
 emf-kuma                                         x86_64                   6.2.0-2.el8                    emf                   8.4 k
 emf-manager                                      x86_64                   6.2.0-2.el8                    emf                   7.9 k
 emf-manager-target                               x86_64                   6.2.0-2.el8                    emf                   8.5 k
 emf-network                                      x86_64                   6.2.0-2.el8                    emf                   2.9 M
 emf-network-agent                                x86_64                   6.2.0-2.el8                    emf                   2.1 M
 emf-network-ib                                   x86_64                   6.2.0-2.el8                    emf                   2.8 M
 emf-nginx                                        x86_64                   6.2.0-2.el8                    emf                    11 k
 emf-node-manager                                 x86_64                   6.2.0-2.el8                    emf                   4.6 M
 emf-ntp                                          x86_64                   6.2.0-2.el8                    emf                   2.1 M
 emf-ntp-agent                                    x86_64                   6.2.0-2.el8                    emf                   2.0 M
 emf-ostpool                                      x86_64                   6.2.0-2.el8                    emf                   2.2 M
 emf-ostpool-agent                                x86_64                   6.2.0-2.el8                    emf                   2.0 M
 emf-perf-cli                                     x86_64                   6.2.0-2.el8                    emf                   4.0 M
 emf-postgres                                     x86_64                   6.2.0-2.el8                    emf                   6.4 k
 emf-resource-agents                              x86_64                   6.2.0-2.el8                    emf                   1.8 M
 emf-sfa                                          x86_64                   6.2.0-2.el8                    emf                   2.7 M
 emf-snapshot                                     x86_64                   6.2.0-2.el8                    emf                   3.3 M
 emf-snapshot-agent                               x86_64                   6.2.0-2.el8                    emf                   2.0 M
 emf-sos-plugin                                   noarch                   6.2.0-10.el8                   emf                    14 k
 emf-state-machine                                x86_64                   6.2.0-2.el8                    emf                   5.5 M
 emf-stats                                        x86_64                   6.2.0-2.el8                    emf                   2.4 M
 emf-stats-agent                                  x86_64                   6.2.0-2.el8                    emf                   2.2 M
 emf-warp-drive                                   x86_64                   6.2.0-2.el8                    emf                   2.4 M

Transaction Summary
======================================================================================================================================
Upgrade  51 Packages

Total download size: 141 M
Is this ok [y/N]:
:
:
:
Upgraded:
  emf-action-agent-6.2.0-2.el8.x86_64                emf-agent-6.2.0-2.el8.x86_64            emf-agent-target-6.2.0-2.el8.x86_64
  emf-api-6.2.0-2.el8.x86_64                         emf-bootstrap-6.2.0-2.el8.x86_64        emf-cli-6.2.0-2.el8.x86_64
  emf-cli-bash-completion-6.2.0-2.el8.x86_64         emf-cluster-cli-6.2.0-2.el8.x86_64      emf-config-cli-6.2.0-2.el8.x86_64
  emf-config-cli-bash-completion-6.2.0-2.el8.x86_64  emf-corosync-6.2.0-2.el8.x86_64         emf-corosync-agent-6.2.0-2.el8.x86_64
  emf-device-6.2.0-2.el8.x86_64                      emf-device-agent-6.2.0-2.el8.x86_64     emf-device-scanner-6.2.0-10.el8.x86_64
  emf-embedded-6.2.0-2.el8.x86_64                    emf-es-fence-agents-6.2.0-2.el8.x86_64  emf-es-tools-6.2.0-2.el8.x86_64
  emf-exa-node-config-6.2.0-2.el8.x86_64             emf-exa-templates-6.2.0-2.el8.x86_64    emf-gpu-6.2.0-2.el8.x86_64
  emf-grafana-6.2.0-2.el8.x86_64                     emf-gui-6.2.0-10.el8.x86_64             emf-host-6.2.0-2.el8.x86_64
  emf-host-agent-6.2.0-2.el8.x86_64                  emf-influx-6.2.0-2.el8.x86_64           emf-journal-6.2.0-2.el8.x86_64
  emf-journal-agent-6.2.0-2.el8.x86_64               emf-kuma-6.2.0-2.el8.x86_64             emf-manager-6.2.0-2.el8.x86_64
  emf-manager-target-6.2.0-2.el8.x86_64              emf-network-6.2.0-2.el8.x86_64          emf-network-agent-6.2.0-2.el8.x86_64
  emf-network-ib-6.2.0-2.el8.x86_64                  emf-nginx-6.2.0-2.el8.x86_64            emf-node-manager-6.2.0-2.el8.x86_64
  emf-ntp-6.2.0-2.el8.x86_64                         emf-ntp-agent-6.2.0-2.el8.x86_64        emf-ostpool-6.2.0-2.el8.x86_64
  emf-ostpool-agent-6.2.0-2.el8.x86_64               emf-perf-cli-6.2.0-2.el8.x86_64         emf-postgres-6.2.0-2.el8.x86_64
  emf-resource-agents-6.2.0-2.el8.x86_64             emf-sfa-6.2.0-2.el8.x86_64              emf-snapshot-6.2.0-2.el8.x86_64
  emf-snapshot-agent-6.2.0-2.el8.x86_64              emf-sos-plugin-6.2.0-10.el8.noarch      emf-state-machine-6.2.0-2.el8.x86_64
  emf-stats-6.2.0-2.el8.x86_64                       emf-stats-agent-6.2.0-2.el8.x86_64      emf-warp-drive-6.2.0-2.el8.x86_64

Complete!
```
- Disable back EMF repo:
```
# vi /etc/yum.repos.d/emf.repo
# grep enabled /etc/yum.repos.d/emf.repo
enabled=0
```
- Unmanage EMF services from cluster before restarting:
``` 
# emfctl unmanage
✔ EMF Services now unmanaged
```
- Restart EMF services:
```
# emfctl stop
# emfctl start
```
- And manage back EMF services under cluster:
```
# emfctl manage
✔ EMF Services now managed
```
- Repeat above steps for rest of VMs, excluding steps for EMF restart:
```
# dnf install emf-repo-rpm-6.2.0-2.el8.x86_64.rpm
# dnf install emf-repo-deb-6.2.0-2.el8.x86_64.rpm
# dnf repolist all
# vi /etc/yum.repos.d/emf.repo ### Enable EMF repo
# dnf repolist all
# dnf update
# vi /etc/yum.repos.d/emf.repo ### Disable back EMF repo
# dnf repolist all
```
