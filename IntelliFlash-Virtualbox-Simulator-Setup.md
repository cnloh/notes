1. Download iso image from http://s1.tegile.com/ps/fw
   - Example: http://s1.tegile.com/ps/fw/Aachen/Release-SW/Intelliflash-3_11_4_1.4.iso

2. Configure two VM for HA pair setup. Sample VM configuration:
   - OS Type: OpenSolaris (64-bit)
   - vCPUs: 4
   - Memory: 12GB (Tested with 8GB and below -> VM will hang during cluster startup)
   - ISO Image: Intelliflash-3_11_4_1.4.iso
   - Root Disk: 30GB
   - HBA for Data Disks: AHCI (SATA) (Tried SAS controller type but assigned vdisks will not be recognised by ZebiOS)
   - Data Disk: Starts with one VDH, any size and must be configured as shareable. To be assigned as quorum disk during cluster configuration.
   - NIC: Assign two network adapter. One for mgmt while the other for cluster interconnect.
     ![](./Images/intelliflash-sample-vm-config.png)

3. Proceed to boot from the IntelliFlash ISO image:  
   ![](./Images/intellisflash-initial-os-install01.png)

4. After booting, follow onscreen instructions to install IntelliFlash base OS (ZebiOS) and software:
   ![](./Images/intellisflash-initial-os-install02.png)
   ![](./Images/intellisflash-initial-os-install03.png)
   ![](./Images/intellisflash-initial-os-install04.png)

5. Reboot for the freshly installed system to take effect:
   ![](./Images/intellisflash-initial-os-install05.png)

6. Once system rebooted, login as root / root123:
   ![](./Images/intellisflash-initial-os-install06.png)

7. Before proceeding with the "zebiconfig.sh" configuration script to perform initial setup, verify the device name to assign for quorum, mgmt and cluster interconnect interfaces.
   - To check on the quorum disk to assign:
     ```
     [root@intelliflash:~]# echo | format
     Searching for disks...done

     AVAILABLE DISK SELECTIONS:
            0. c1d0 <Unknown-Unknown-0001 cyl 6523 alt 2 hd 255 sec 63>
               /pci@0,0/pci-ide@1,1/ide@0/cmdk@0,0
            1. c2t0d0 <VBOX-HARDDISK-1.0-1.00GB>
               /pci@0,0/pci8086,2829@d/disk@0,0
     Specify disk (enter its number): Specify disk (enter its number):
     ```
     From above ouput, "c2t0d0" shall be the disk to be assigned as the quorum disk.
   - To check on the network interfaces to assign:
     ```
     [root@intelliflash:~]# dladm show-link
     LINK        CLASS     MTU    STATE    BRIDGE     OVER
     e1000g0     phys      1500   unknown       --         --
     e1000g1     phys      1500   unknown  --         --
     ```
     e1000g0 -> Will be assigned for mgmt interface.  
     e1000g1 -> Will be assigned for cluster interconnect.  

8. Kick start the initial configuration script "zebiconfig.sh".
    - Initial screen:
      ![](./Images/intellisflash-initial-config-01.png)
    - Enter hostname and domainname for controller:
      ![](./Images/intellisflash-initial-config-02.png)
    - 'Y' to configure as HA pair and enter the quorum and interconnect device name as capture in step (7):
      ![](./Images/intellisflash-initial-config-03.png) 
    - Time zone configuration:
      ![](./Images/intellisflash-initial-config-04.png)
      ![](./Images/intellisflash-initial-config-05.png)
      ![](./Images/intellisflash-initial-config-06.png)
      ![](./Images/intellisflash-initial-config-07.png)
      ![](./Images/intellisflash-initial-config-08.png)
    - Setting up the password for 'zebiadmin', 'root' & 'admin':
      ![](./Images/intellisflash-initial-config-09.png)
    - Answer 'N' for NDMP, IntelliCare & KVM settings.
    - And complete the rest of configuration settings as prompted:
      ![](./Images/intellisflash-initial-config-10.png)
    - Reboot the system when prompted.
9. Upon system rebooted, web console will be the recommended interface to manage it:
   ![]/(./Images/intellisflash-web-console-login.png)
10. 
11. 
12. 
13. 
14. 
15. 
16. 
17. 
18. 
19. 
20. 
21. 
22. 
23. 
   
