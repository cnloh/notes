- Copy the client software package from Lustre server:
  ```
  # scp 192.168.2.101:/scratch/EXAScaler-6.1.0/exa-client-6.1.0.tar.gz .
  ```

- Unzip the downloaded package:
  ```
  # tar xf exa-client-6.1.0.tar.gz
  ```

- Run the setup script and select option 2 to install the software (internet access required or client has been configured for offline Ubuntu software repositories) :
  ```
  # cd exa-client
  # ./exa_client_deploy.py

  DDN EXAScaler client software installation tool: Version 6.1.0
  Select an option:

  1) Check if DDN EXAScaler client software is installed
  2) Install DDN EXAScaler client software
  3) Configure DDN EXAScaler client software
  4) Remove DDN EXAScaler client software
  5) Exit
  2

  Mellanox OFED is not installed. Lustre will be built against in-kernel IB stack.
  Do you want to continue [Y/n]: Y

  Selected /root/exa-client/lustre-source.tar.gz for installation

  Preparing build environment...
  Preparing build environment... Done

  Building EXAScaler client software. This may take a while...
  EXAScaler client software packages are installed and placed in /opt/ddn/exascaler/debs folder
  Use option 3 to configure EXAScaler client software before loading lustre module
  ```

- Next, proceed to used option 3 to configure the software as instructed.  
  Do take note on the LNET net name (take reference from Lustre server) and the data interface (enp0s8 in below example) that will be used to connect to the Lustre filesystem.  
  Data interface should already have IP address assigned and configured.  
  ```
  DDN EXAScaler client software installation tool: Version 6.1.0
  Select an option:

  1) Check if DDN EXAScaler client software is installed
  2) Install DDN EXAScaler client software
  3) Configure DDN EXAScaler client software
  4) Remove DDN EXAScaler client software
  5) Exit
  3
  Specify LNets (semicolon separated) - e.g. [o2ib(ens1,ens2)]: tcp(enp0s8)

  Apply Ethernet tunings [Y/n]: Y
  Applying Ethernet tuning (ETA: <1 minutes)

  EXAScaler client software is configured

  DDN EXAScaler client software installation tool: Version 6.1.0
  Select an option:

  1) Check if DDN EXAScaler client software is installed
  2) Install DDN EXAScaler client software
  3) Configure DDN EXAScaler client software
  4) Remove DDN EXAScaler client software
  5) Exit
  5
  ```

- Load and enable the LNET service for the client:
  ```
  # modprobe lnet
  # systemctl enable lnet.service
  Created symlink /etc/systemd/system/multi-user.target.wants/lnet.service → /lib/systemd/system/lnet.service.
  ```

- Load the Lustre module:
  ```
  # modprobe lustre
  ```

- Check connectivity with server:
  ```
  # lctl list_nids
  192.168.3.117@tcp
  # lctl ping 192.168.3.105@tcp
  12345-0@lo
  12345-192.168.3.105@tcp
  12345-192.168.3.109@tcp
  ```

- Mounting the EXAScaler filesystem:
  ```
  # mkdir /mnt/testfs
  # mount -t lustre  192.168.3.105@tcp,192.168.3.109@tcp:/testfs /mnt/testfs
  ```

- Sample /etc/fstab configuration for automated filesystem mount:
  ```
  # tail -n 1 /etc/fstab
  192.168.3.105@tcp,192.168.3.109@tcp:/testfs /mnt/testfs lustre defaults,_netdev 0 0
  ```
