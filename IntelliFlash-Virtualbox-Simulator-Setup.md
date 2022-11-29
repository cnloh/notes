1. Download iso image from http://s1.tegile.com/ps/fw
   - Example: http://s1.tegile.com/ps/fw/Aachen/Release-SW/Intelliflash-3_11_4_1.4.iso
2. Configure two VM for HA pair setup. Recommended to configure each VM with at 12GB of memory.  Sample VM configuration:
   - OS Type: OpenSolaris (64-bit)
   - vCPUs: 4
   - Memory: 12GB (Tested with 8GB and below -> VM will hang during cluster startup)
   - Root Disk: 20GB
   - HBA for Data Disks: AHCI (SATA) (Tried SAS controller type but assigned vdisks will not be recognised by ZebiOS)
   - Data Disk: Starts with one VDH, any size. To be assigned as quorum disk during cluster configuration.
   - NIC: Assign two network adapter. One for mgmt while the other for cluster interconnect.
   [](./Images/sample-vm-config.png)
 3. 
