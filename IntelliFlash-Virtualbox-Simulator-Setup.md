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
9. Upon system rebooted, web console will be available to manage the system.
   - Web console login:  
     ![]/(./Images/intellisflash-web-console-login.png)
   - To view the allocated disk(s), from top menu navigate to Settings -> Hardware:
     ![]/(./Images/intellisflash-web-console-disk-hdd.png)
   - To "trick" system to recognize the disks as SSD instead (since this is just a simulator setup), we can edit the disks' model configuration file ```/opt/tomcat/webapps/zebi/model/ssdmodel.xml```. Steps:
     - Establish ssh session to the VM as "zebiadmin" and "su -" with root access:
       ```
       -bash-4.4$ ssh zebiadmin@ctrl-a
       (zebiadmin@ctrl-a) Password:

       Welcome to Tintri IntelliFlash.
       Access to the system console is restricted. If you are not authorized
       to access the system console, please logout and leave immediately.

       -bash-4.4$ su -
       Password:

       Welcome to Tintri IntelliFlash.
       Access to the system console is restricted. If you are not authorized
       to access the system console, please logout and leave immediately.

        [root@ctrl-a:~]#
       ```
     - Check assigned virtual disk's vendor and product type:
       ```
       [root@ctrl-a:~]# echo | format
       Searching for disks...done

       AVAILABLE DISK SELECTIONS:
              0. c1d0 <Unknown-Unknown-0001 cyl 6523 alt 2 hd 255 sec 63>
                 /pci@0,0/pci-ide@1,1/ide@0/cmdk@0,0
              1. c2t0d0 <VBOX-HARDDISK-1.0-1.00GB>
                 /pci@0,0/pci8086,2829@d/disk@0,0
       Specify disk (enter its number): Specify disk (enter its number):
       [root@ctrl-a:~]# sg_inq /dev/rdsk/c2t0d0s2 | grep Product
        Product identification: VBOX HARDDISK
        Product revision level: 1.0
       [root@ctrl-a:~]#
       ```
     - Add an entry in ```/opt/tomcat/webapps/zebi/model/ssdmodel.xml``` (contents with this xml file is self explanatory) to recognize "VBOX" "HARDDISK" as flash disk. Sample "VBOX" entry added:
       ```
       [root@ctrl-a:/]# egrep "flash|VBOX" /opt/tomcat/webapps/zebi/model/ssdmodel.xml
               <!-- empty bias means it's for general data (all-flash) purpose -->
               <model name="HARDDISK" vendor="VBOX" bias="" />
       ```
     - Restart the web console service (tomcat) to take effect.
       ```
       [root@ctrl-a:/]# svccadm restart tomcat
       [root@ctrl-a:/]# svcs tomcat
       STATE          STIME    FMRI
       online*        16:48:48 svc:/network/tomcat:default
       ```
     - From web console, navigate back to Settings -> Hardware section and you will find the data disk has been recognized as SSD:
       ![]/(./Images/intellisflash-web-console-disk-ssd.png)
10. Repeat above step (2) to (9) to setup VM for "ctrl-b". Things to take note:
    - Assign the same quorum disk (c2t0d0 as shown above) to this VM as the name's function imply.
    - IMPORTANT - At step (8), before executing "zebiconfig.sh" script please do edit below indicated environment profile scripts first.
      Before changes:
      ```
      # grep CONTROLLER_NO /etc/zebi/env.bash
      export CONTROLLER_NO="0"
      # grep CONTROLLER_NO /etc/zebi/env.sh
      CONTROLLER_NO="0"
      ```  
      After changes:
      ```
      # grep CONTROLLER_NO /etc/zebi/env.bash
      export CONTROLLER_NO="1"
      # grep CONTROLLER_NO /etc/zebi/env.sh
      CONTROLLER_NO="1"
      ```  
      Above changes will indicate this VM as the second controller else there will be issues when setting up the HA pair as both will think they are the first controller 0.
11. Verifying correct VM's node assignment.
    - Establish ssh session to each VM:
      - From VM designated as Controller A:
        ```
        [root@ctrl-a:~]# cat /etc/hanodename
        ha-controller-a
        ```
      - From VM designated as Controller B:
        ```
        [root@ctrl-b:~]# cat /etc/hanodename
        ha-controller-b
        ```
12. Establish HA Pair. 
    - Login to ctrl-b web console. From top menu, Settings -> High Availability
      ![]/(./Images/intellisflash-config-ha01.png)
    - Click "Configure HA" and you will be prompted to enter peer node (ctrl-a) login (admin) credential. 
      ![]/(./Images/intellisflash-config-ha02.png)
    - Configuration of peer node (ctrl-a) will start and reboot. ctrl-b will follow next automatically after ctrl-a has rebooted into cluster mode.
      ![]/(./Images/intellisflash-config-ha03.png)
    - It will take some time for both nodes to be configured into HA pair. When HA has been established, you will only be able to login to the web console of ctrl-a. After login in as admin, you will be presented with the Initial Configuration Wizard (ICW) page:
      ![]/(./Images/intellisflash-config-ha04.png)
    - For unknown reason, you will not be able to proceed any further with the ICW as whatever settings entered, can't be saved (click "save" but nothing will happen). However that will not be a concern as we can always configure the rest of settings manually. 
13. 
14. 
15. 
16. 
17. 
18. 
20. 
21. 
22. 
23. 
24. 
25. 
   
