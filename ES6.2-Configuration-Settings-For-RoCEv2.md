#### TERMS:
- L3 Differentiated Serviced Code Point (DSCP) -> Think of it as a "Priority Sticker" you put on a packet
- L2 Priority Code Points (PCP)
- Priority-Flow-Control (PFC) -> "brake" mechanism
- Explicit Congestion Notification packets (ECN) -> "speed limit" or "transmission rates reduction" mechanism
- ECN that are received are handled by the Data Center Quantized Congestion Notification Scheme (DCQCN)
- RoCEv2 Congestion Notification Packet (CNP)

#### 1) Check RoCE version enabled:
```
# mst start
# mst status -v
MST modules:
------------
    MST PCI module is not loaded
    MST PCI configuration module loaded
PCI devices:
------------
DEVICE_TYPE             MST                           PCI       RDMA            NET                       NUMA
ConnectX6(rev:0)        /dev/mst/mt4123_pciconf1      02:00.0   mlx5_1          net-mlxen1                0
ConnectX6(rev:0)        /dev/mst/mt4123_pciconf0      01:00.0   mlx5_0          net-mlxen0                0

# ibdev2netdev
mlx5_0 port 1 ==> mlxen0 (Up)
mlx5_1 port 1 ==> mlxen1 (Up)

# cma_roce_mode -d mlx5_0
RoCE v2
# cma_roce_mode -d mlx5_1
RoCE v2
```
#### 2) To configures a lossless network for RoCEv2 to operate on:
```
# cat /etc/NetworkManager/dispatcher.d/10-ifup-local
#!/bin/bash
#
# Script to apply lossless-RoCE QoS settings for storage target 100Gb interfaces #
case $1 in
  mlxen0)
    if [ "$2" == "up" ]; then
      echo "Configuring DCBX PFC control, QoS L3-trust (DSCP) and RoCE ToS for mlxen0"
      /usr/bin/mlnx_qos -i mlxen0 --trust dscp
      /usr/bin/mlnx_qos -i mlxen0 -f 0,0,0,1,0,0,0,0
      /usr/sbin/cma_roce_tos -d mlx5_0 -t 106
    fi
    ;;
  mlxen1)
    if [ "$2" == "up" ]; then
    echo "Configuring DCBX PFC control, QoS L3-trust (DSCP) and RoCE ToS for mlxen1"
    /usr/bin/mlnx_qos -i mlxen1 --trust dscp
    /usr/bin/mlnx_qos -i mlxen1 -f 0,0,0,1,0,0,0,0
    /usr/sbin/cma_roce_tos -d mlx5_1 -t 106
    fi
    ;;
esac
exit 0
# chmod 755 /etc/NetworkManager/dispatcher.d/10-ifup-local
# reboot
```
To verify changes in place:
```
# mlnx_qos -i mlxen0
# mlnx_qos -i mlxen1
```
#### 3) Verify Global Pause is no longer enabled (make sure you disable it on the switch as well):
```
# ethtool -a mlxen0
# ethtool -a mlxen1
Pause parameters for mlxen1:
Autonegotiate:  off
RX:             off
TX:             off
```
#### 4) To configure RoCEv2 Congestion-Control, execute the shown echo output from below last command: 
```
# IF=mlxen0
# mst start
# device=$(mst status -v |grep ${IF} |awk '{print $2}'|head -n 1)
# port=$(mlxconfig -d ${device} query |grep LINK_TYPE|grep ETH|grep -E -o "P[0-9]" |grep -E -o "[0-9]"|head -n1)
# mlxconfig -d ${device} query | egrep "ROCE_CC_PRIO_MASK_P${port}|RPG_THRESHOLD_P${port}|DCE_TCP_G_P${port}|LLDP_NB_DCBX_P${port}|LLDP_NB_TX_MODE_P${port}|LLDP_NB_RX_MODE_P${port}|CNP_802P_PRIO_P${port}"
         ROCE_CC_PRIO_MASK_P1                        255
         RPG_THRESHOLD_P1                            1
         DCE_TCP_G_P1                                1019
         CNP_802P_PRIO_P1                            6
         LLDP_NB_DCBX_P1                             False(0)
         LLDP_NB_RX_MODE_P1                          OFF(0)
         LLDP_NB_TX_MODE_P1                          OFF(0)
# echo "mlxconfig -d ${device} -y set ROCE_CC_PRIO_MASK_P${port}=255 RPG_THRESHOLD_P${port}=1 DCE_TCP_G_P${port}=1019 LLDP_NB_DCBX_P${port}=TRUE LLDP_NB_TX_MODE_P${port}=2 LLDP_NB_RX_MODE_P${port}=2 CNP_802P_PRIO_P${port}=6"
mlxconfig -d /dev/mst/mt4123_pciconf0 -y set ROCE_CC_PRIO_MASK_P1=255 RPG_THRESHOLD_P1=1 DCE_TCP_G_P1=1019 LLDP_NB_DCBX_P1=TRUE LLDP_NB_TX_MODE_P1=2 LLDP_NB_RX_MODE_P1=2 CNP_802P_PRIO_P1=6
```
To verify the changes:
```
# mlxconfig -d ${device} query | egrep "ROCE_CC_PRIO_MASK_P${port}|RPG_THRESHOLD_P${port}|DCE_TCP_G_P${port}|LLDP_NB_DCBX_P${port}|LLDP_NB_TX_MODE_P${port}|LLDP_NB_RX_MODE_P${port}|CNP_802P_PRIO_P${port}"
         ROCE_CC_PRIO_MASK_P1                        255
         RPG_THRESHOLD_P1                            1
         DCE_TCP_G_P1                                1019
         CNP_802P_PRIO_P1                            6
         LLDP_NB_DCBX_P1                             False(0)  ==> Change to True(1)
         LLDP_NB_RX_MODE_P1                          OFF(0)    ==> Change to ALL(2)
         LLDP_NB_TX_MODE_P1                          OFF(0)    ==> Change to ALL(2)
```
Reboot to commit changes.
#### 6) Above steps are applicable for Ubuntu client with Nvidia ConnectX NIC installed. However the "10-ifup-local" hook script will have to be placed in a different directory and with modification as shown below (do edit the interface name accordingly):
```
# mkdir /etc/networkd-dispatcher/configured.d
# vi /etc/networkd-dispatcher/configured.d/10-ifup-local
# cat /etc/networkd-dispatcher/configured.d/10-ifup-local
#!/bin/bash
#
# Script to apply lossless-RoCE QoS settings for storage target 100Gb interfaces #
case $IFACE in
  enp170s0f0np0)
    echo "Configuring DCBX PFC control, QoS L3-trust (DSCP) and RoCE ToS for enp170s0f0np0"
    /usr/bin/mlnx_qos -i enp170s0f0np0 --trust dscp
    /usr/bin/mlnx_qos -i enp170s0f0np0 -f 0,0,0,1,0,0,0,0
    /usr/sbin/cma_roce_tos -d mlx5_7 -t 106
    ;;
  enp41s0f0np0)
    echo "Configuring DCBX PFC control, QoS L3-trust (DSCP) and RoCE ToS for enp41s0f0np0"
    /usr/bin/mlnx_qos -i enp41s0f0np0 --trust dscp
    /usr/bin/mlnx_qos -i enp41s0f0np0 -f 0,0,0,1,0,0,0,0
    /usr/sbin/cma_roce_tos -d mlx5_1 -t 106
    ;;
esac
exit 0
# chmod 755 /etc/networkd-dispatcher/configured.d/10-ifup-local
```
#### 7) Verification. During performance testing following counters should have increased, indicating server end is both sending and receiving ECN packets:
```
# cat /sys/class/infiniband/mlx5_*/ports/1/hw_counters/np_cnp_sent
3773741
4232220
# cat /sys/class/infiniband/mlx5_*/ports/1/hw_counters/np_ecn_marked_roce_packets
79402419
112641514
```
