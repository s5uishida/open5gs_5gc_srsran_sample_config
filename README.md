# Open5GS 5GC & srsRAN 5G with ZeroMQ UE / RAN Sample Configuration
srsRAN_Project and srsRAN_4G software suites include a virtual radio which uses the ZeroMQ networking library to transfer radio samples between applications.
Therefore, in order to use U-Plane's DN (Data Network) as a trial, I built a simulation environment for the 5GC mobile network.
This briefly describes the overall and configuration files in the Virtualbox VM environment.

---

### [Sample Configurations and Miscellaneous for Mobile Network](https://github.com/s5uishida/sample_config_misc_for_mobile_network)

---

<a id="toc"></a>

## Table of Contents

- [Overview of Open5GS 5GC Simulation Mobile Network](#overview)
- [Changes in configuration files of Open5GS 5GC and srsRAN 5G ZMQ UE / RAN](#changes)
  - [Changes in configuration files of Open5GS 5GC C-Plane](#changes_cp)
  - [Changes in configuration files of Open5GS 5GC U-Plane](#changes_up)
  - [Changes in configuration files of srsRAN 5G ZMQ UE / RAN](#changes_srs)
    - [Changes in configuration files of RAN](#changes_ran)
    - [Changes in configuration files of UE](#changes_ue)
- [Network settings of Open5GS 5GC](#network_settings)
  - [Network settings of Open5GS 5GC U-Plane](#network_settings_up)
- [Build Open5GS and srsRAN 5G ZMQ UE / RAN](#build)
- [Run Open5GS 5GC and srsRAN 5G ZMQ UE / RAN](#run)
  - [Run Open5GS 5GC C-Plane](#run_cp)
  - [Run Open5GS 5GC U-Plane](#run_up)
  - [Run srsRAN 5G ZMQ RAN](#run_ran)
  - [Run srsRAN 5G ZMQ UE](#run_ue)
- [Ping google.com](#ping)
  - [Case for going through DN 10.45.0.0/16](#ping_1)
- [Changelog (summary)](#changelog)
---

<a id="overview"></a>

## Overview of Open5GS 5GC Simulation Mobile Network

I created a 5GC mobile network (Internet reachable) for simulation with the aim of creating an environment in which packets can be sent end-to-end with one DN for one DNN.

The following minimum configuration was set as a condition.
- Only one each for C-Plane, U-Plane and UE.

The built simulation environment is as follows.

<img src="./images/network-overview.png" title="./images/network-overview.png" width=1000px></img>

The 5GC / UE / RAN used are as follows.
- 5GC - Open5GS v2.7.2 (2024.10.18) - https://github.com/open5gs/open5gs
- RAN - srsRAN Project (2024.10.14) - https://github.com/srsran/srsRAN_Project
- UE (NR-UE) - srsRAN 4G (2024.02.01) - https://github.com/srsran/srsRAN_4G

Each VMs are as follows.  
| VM | SW & Role | IP address | OS | CPU <br> (Min) | Memory <br> (Min) | HDD <br> (Min) |
| --- | --- | --- | --- | --- | --- | --- |
| VM1 | Open5GS 5GC C-Plane | 192.168.0.111/24 | Ubuntu 24.04 | 1 | 2GB | 20GB |
| VM2 | Open5GS 5GC U-Plane  | 192.168.0.112/24 | Ubuntu 24.04 | 1 | 1GB | 20GB |
| VM3 | srsRAN Project ZMQ RAN<br>(gNodeB) | 192.168.0.121/24 | Ubuntu 24.04 | 2 | 4GB | 10GB |
| VM4 | srsRAN 4G ZMQ UE<br>(NR-UE) | 192.168.0.122/24 | Ubuntu 22.04 | 1 | 2GB | 10GB |

AMF & SMF addresses are as follows.  
| NF | IP address | IP address on SBI | Supported S-NSSAI |
| --- | --- | --- | --- |
| AMF | 192.168.0.111 | 127.0.0.5 | SST:1, SD:0x000001 |
| SMF | 192.168.0.111 | 127.0.0.4 | SST:1, SD:0x000001 |

Subscriber Information (other information is the same) is as follows.  
| UE | IMSI | DNN | OP/OPc | S-NSSAI |
| --- | --- | --- | --- | --- |
| UE (NR-UE) | 001010000000000 | internet | OPc | SST:1, SD:0x000001 |

I registered these information with the Open5GS WebUI.
In addition, [3GPP TS 35.208](https://www.3gpp.org/DynaReport/35208.htm) "4.3 Test Sets" is published by 3GPP as test data for the 3GPP authentication and key generation functions (MILENAGE).

DN is as follows.
| DN | S-NSSAI | TUNnel interface of DN | DNN | TUNnel interface of UE |
| --- | --- | --- | --- | --- |
| 10.45.0.0/16 | SST:1 <br> SD:0x000001 | ogstun | internet | tun_srsue |

The main information of gNodeB is as follows.
| MCC | MNC | TAC | gNodeB ID |
| --- | --- | --- | --- |
| 001 | 01 | 1 | 0x19b |

Additional information.

Open5GS 5GC U-Plane worked fine on Raspberry Pi 4 Model B. I used [Ubuntu 20.04 (64bit) for Raspberry Pi 4](https://ubuntu.com/download/raspberry-pi) as the OS. I think it would be convenient to place a compact U-Plane in the edge environment and use it as an end-point for DN.

In addition, I have not confirmed the communication performance.

<a id="changes"></a>

## Changes in configuration files of Open5GS 5GC and srsRAN 5G ZMQ UE / RAN

Please refer to the following for building Open5GS and srsRAN 5G ZMQ UE / RAN respectively.
- Open5GS v2.7.2 (2024.10.18) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- srsRAN Project (RAN) (2024.10.14) - https://github.com/s5uishida/build_srsran_5g_zmq
- srsRAN 4G (UE) (2024.02.01) - https://github.com/s5uishida/build_srsran_4g_zmq_disable_rf_plugins

<a id="changes_cp"></a>

### Changes in configuration files of Open5GS 5GC C-Plane

The following parameters can be used in the logic that selects UPF as the connection destination by PFCP.

- DNN
- TAC (Tracking Area Code)
- nr_CellID

For the sake of simplicity, I used only DNN this time.

<a id="changes_cp"></a>

### Changes in configuration files of Open5GS 5GC C-Plane

- `open5gs/install/etc/open5gs/amf.yaml`
```diff
--- amf.yaml.orig       2024-03-24 15:36:48.000000000 +0900
+++ amf.yaml    2024-03-29 22:14:08.359119294 +0900
@@ -19,29 +19,30 @@
         - uri: http://127.0.0.200:7777
   ngap:
     server:
-      - address: 127.0.0.5
+      - address: 192.168.0.111
   metrics:
     server:
       - address: 127.0.0.5
         port: 9090
   guami:
     - plmn_id:
-        mcc: 999
-        mnc: 70
+        mcc: 001
+        mnc: 01
       amf_id:
         region: 2
         set: 1
   tai:
     - plmn_id:
-        mcc: 999
-        mnc: 70
+        mcc: 001
+        mnc: 01
       tac: 1
   plmn_support:
     - plmn_id:
-        mcc: 999
-        mnc: 70
+        mcc: 001
+        mnc: 01
       s_nssai:
         - sst: 1
+          sd: 000001
   security:
     integrity_order : [ NIA2, NIA1, NIA0 ]
     ciphering_order : [ NEA0, NEA1, NEA2 ]
```
- `open5gs/install/etc/open5gs/nrf.yaml`
```diff
--- nrf.yaml.orig       2024-03-24 15:36:48.000000000 +0900
+++ nrf.yaml    2024-03-29 22:38:40.866371099 +0900
@@ -10,8 +10,8 @@
 nrf:
   serving:  # 5G roaming requires PLMN in NRF
     - plmn_id:
-        mcc: 999
-        mnc: 70
+        mcc: 001
+        mnc: 01
   sbi:
     server:
       - address: 127.0.0.10
```
- `open5gs/install/etc/open5gs/smf.yaml`
```diff
--- smf.yaml.orig       2024-03-24 15:36:48.000000000 +0900
+++ smf.yaml    2024-03-31 22:45:18.892357262 +0900
@@ -19,35 +19,31 @@
         - uri: http://127.0.0.200:7777
   pfcp:
     server:
-      - address: 127.0.0.4
+      - address: 192.168.0.111
     client:
       upf:
-        - address: 127.0.0.7
-  gtpc:
-    server:
-      - address: 127.0.0.4
+        - address: 192.168.0.112
+          dnn: internet
   gtpu:
     server:
-      - address: 127.0.0.4
+      - address: 192.168.0.111
   metrics:
     server:
       - address: 127.0.0.4
         port: 9090
   session:
     - subnet: 10.45.0.1/16
-    - subnet: 2001:db8:cafe::1/48
+      dnn: internet
   dns:
     - 8.8.8.8
     - 8.8.4.4
-    - 2001:4860:4860::8888
-    - 2001:4860:4860::8844
   mtu: 1400
 #  p-cscf:
 #    - 127.0.0.1
 #    - ::1
 #  ctf:
 #    enabled: auto   # auto(default)|yes|no
-  freeDiameter: /root/open5gs/install/etc/freeDiameter/smf.conf
+#  freeDiameter: /root/open5gs/install/etc/freeDiameter/smf.conf
 
 ################################################################################
 # SMF Info
```

<a id="changes_up"></a>

### Changes in configuration files of Open5GS 5GC U-Plane

- `open5gs/install/etc/open5gs/upf.yaml`
```diff
--- upf.yaml.orig       2024-03-24 15:36:48.000000000 +0900
+++ upf.yaml    2024-03-29 22:18:51.683148805 +0900
@@ -10,16 +10,17 @@
 upf:
   pfcp:
     server:
-      - address: 127.0.0.7
+      - address: 192.168.0.112
     client:
 #      smf:     #  UPF PFCP Client try to associate SMF PFCP Server
 #        - address: 127.0.0.4
   gtpu:
     server:
-      - address: 127.0.0.7
+      - address: 192.168.0.112
   session:
     - subnet: 10.45.0.1/16
-    - subnet: 2001:db8:cafe::1/48
+      dnn: internet
+      dev: ogstun
   metrics:
     server:
       - address: 127.0.0.7
```

<a id="changes_srs"></a>

### Changes in configuration files of srsRAN 5G ZMQ UE / RAN

<a id="changes_ran"></a>

#### Changes in configuration files of RAN

See [here](https://github.com/s5uishida/build_srsran_5g_zmq#create-the-configuration-file-of-gnodeb) for the original files.

- `srsRAN_Project/build/apps/gnb/gnb_zmq.yaml`
```diff
--- gnb_zmq.yaml.orig   2024-10-14 18:46:45.000000000 +0900
+++ gnb_zmq.yaml        2024-11-08 23:43:10.537903842 +0900
@@ -3,22 +3,33 @@
 # To run the srsRAN Project gNB with this config, use the following command: 
 #   sudo ./gnb -c gnb_zmq.yaml
 
+gnb_id: 0x19B
+
+cu_up:
+  upf:
+    bind_addr: 192.168.0.121              # Optional TEXT. Sets local IP address to bind for N3 interface. Format: IPV4 or IPV6 IP address.
+    ext_addr: auto                        # Optional TEXT. Sets external IP address that is advertised to receive GTP-U packets from UPF via N3 interface. Format: IPV4 or IPV6 IP address.
+    udp_max_rx_msgs: 256                  # Optional INT. Sets the maximum amount of messages RX in a single syscall.
+    pool_threshold: 0.9                   # Optional FLOAT. Sets the pool accupancy threshold after which packets are dropped. Supported: [0.0 - 1.0].
+    no_core: false                        # Optional BOOLEAN. Setting to true allows the gNB to run without a core. Supported: [0, 1].
+
 cu_cp:
   amf:
-    addr: 10.53.1.2                 # The address or hostname of the AMF.
+    addr: 192.168.0.111                 # The address or hostname of the AMF.
     port: 38412
-    bind_addr: 10.53.1.1            # A local IP that the gNB binds to for traffic from the AMF.
+    bind_addr: 192.168.0.121            # A local IP that the gNB binds to for traffic from the AMF.
     supported_tracking_areas:
-      - tac: 7
+      - tac: 1
         plmn_list:
           - plmn: "00101"
             tai_slice_support_list:
               - sst: 1
+                sd: 1
   inactivity_timer: 7200            # Sets the UE/PDU Session/DRB inactivity timer to 7200 seconds. Supported: [1 - 7200].
 
 ru_sdr:
   device_driver: zmq                # The RF driver name.
-  device_args: tx_port=tcp://127.0.0.1:2000,rx_port=tcp://127.0.0.1:2001,base_srate=23.04e6 # Optionally pass arguments to the selected RF driver.
+  device_args: tx_port=tcp://192.168.0.121:2000,rx_port=tcp://192.168.0.122:2001,base_srate=23.04e6 # Optionally pass arguments to the selected RF driver.
   srate: 23.04                      # RF sample rate might need to be adjusted according to selected bandwidth.
   tx_gain: 75                       # Transmit gain of the RF might need to adjusted to the given situation.
   rx_gain: 75                       # Receive gain of the RF might need to adjusted to the given situation.
@@ -29,7 +40,7 @@
   channel_bandwidth_MHz: 20         # Bandwith in MHz. Number of PRBs will be automatically derived.
   common_scs: 15                    # Subcarrier spacing in kHz used for data.
   plmn: "00101"                     # PLMN broadcasted by the gNB.
-  tac: 7                            # Tracking area code (needs to match the core configuration).
+  tac: 1                            # Tracking area code (needs to match the core configuration).
   pdcch:
     common:
       ss0_index: 0                  # Set search space zero index to match srsUE capabilities
```

<a id="changes_ue"></a>

#### Changes in configuration files of UE

See [here](https://github.com/s5uishida/build_srsran_4g_zmq_disable_rf_plugins#create-the-configuration-file-of-nr-ue) for the original files.

- `srsRAN_4G/build/srsue/ue_zmq.conf`
```diff
--- ue_zmq.conf.orig    2023-12-07 03:04:57.000000000 +0900
+++ ue_zmq.conf 2024-03-29 22:41:01.107838116 +0900
@@ -6,7 +6,7 @@
 nof_antennas = 1
 
 device_name = zmq
-device_args = tx_port=tcp://127.0.0.1:2001,rx_port=tcp://127.0.0.1:2000,base_srate=23.04e6
+device_args = tx_port=tcp://192.168.0.122:2001,rx_port=tcp://192.168.0.121:2000,base_srate=23.04e6
 
 [rat.eutra]
 dl_earfcn = 2850
@@ -34,9 +34,9 @@
 [usim]
 mode = soft
 algo = milenage
-opc  = 63BFA50EE6523365FF14C1F45F88737D
-k    = 00112233445566778899aabbccddeeff
-imsi = 001010123456780
+opc  = E8ED289DEBA952E4283B54E88E6183CA
+k    = 465B5CE8B199B49FAA5F0A2EE238A6BC
+imsi = 001010000000000
 imei = 353490069873319
 
 [rrc]
@@ -44,11 +44,16 @@
 ue_category = 4
 
 [nas]
-apn = srsapn
+apn = internet
 apn_protocol = ipv4
 
+[slicing]
+enable = true
+nssai-sst = 1
+nssai-sd = 1
+
 [gw]
-netns = ue1
+#netns = ue1
 ip_devname = tun_srsue
 ip_netmask = 255.255.255.0
 
```

<a id="network_settings"></a>

## Network settings of Open5GS 5GC

<a id="network_settings_up"></a>

### Network settings of Open5GS 5GC U-Plane

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, configure the TUNnel interface and NAPT.
```
ip tuntap add name ogstun mode tun
ip addr add 10.45.0.1/16 dev ogstun
ip link set ogstun up

iptables -t nat -A POSTROUTING -s 10.45.0.0/16 ! -o ogstun -j MASQUERADE
```

<a id="build"></a>

## Build Open5GS and srsRAN 5G ZMQ UE / RAN

Please refer to the following for building Open5GS and srsRAN 5G ZMQ UE / RAN respectively.
- Open5GS v2.7.2 (2024.10.18) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- srsRAN Project (RAN) (2024.10.14) - https://github.com/s5uishida/build_srsran_5g_zmq
- srsRAN 4G (UE) (2024.02.01) - https://github.com/s5uishida/build_srsran_4g_zmq_disable_rf_plugins

Install MongoDB on Open5GS 5GC C-Plane machine.
It is not necessary to install MongoDB on Open5GS 5GC U-Plane machines.
[MongoDB Compass](https://www.mongodb.com/products/compass) is a convenient tool to look at the MongoDB database.

<a id="run"></a>

## Run Open5GS 5GC and srsRAN 5G ZMQ UE / RAN

First run the 5GC, then the RAN, and the UE.

<a id="run_cp"></a>

### Run Open5GS 5GC C-Plane

First, run Open5GS 5GC C-Plane.

- Open5GS 5GC C-Plane
```
./install/bin/open5gs-nrfd &
sleep 2
./install/bin/open5gs-scpd &
sleep 2
./install/bin/open5gs-amfd &
sleep 2
./install/bin/open5gs-smfd &
./install/bin/open5gs-ausfd &
./install/bin/open5gs-udmd &
./install/bin/open5gs-udrd &
./install/bin/open5gs-pcfd &
./install/bin/open5gs-nssfd &
./install/bin/open5gs-bsfd &
```

<a id="run_up"></a>

### Run Open5GS 5GC U-Plane

Next, run Open5GS 5GC U-Plane.

- Open5GS 5GC U-Plane
```
./install/bin/open5gs-upfd &
```

<a id="run_ran"></a>

### Run srsRAN 5G ZMQ RAN

Run srsRAN 5G ZMQ RAN and connect to Open5GS 5GC.
```
# cd srsRAN_Project/build/apps/gnb
# ./gnb -c gnb_zmq.yaml

--== srsRAN gNB (commit 9d5dd742a) ==--


The PRACH detector will not meet the performance requirements with the configuration {Format 0, ZCZ 0, SCS 1.25kHz, Rx ports 1}.
Lower PHY in executor blocking mode.
Available radio types: zmq.
Cell pci=1, bw=20 MHz, 1T1R, dl_arfcn=368500 (n3), dl_freq=1842.5 MHz, dl_ssb_arfcn=368410, ul_freq=1747.5 MHz

N2: Connection to AMF on 192.168.0.111:38412 completed
==== gNB started ===
Type <h> to view help
```
The Open5GS C-Plane log when executed is as follows.
```
11/08 23:54:56.117: [amf] INFO: gNB-N2 accepted[192.168.0.121]:48605 in ng-path module (../src/amf/ngap-sctp.c:113)
11/08 23:54:56.117: [amf] INFO: gNB-N2 accepted[192.168.0.121] in master_sm module (../src/amf/amf-sm.c:813)
11/08 23:54:56.123: [amf] INFO: [Added] Number of gNBs is now 1 (../src/amf/context.c:1240)
11/08 23:54:56.123: [amf] INFO: gNB-N2[192.168.0.121] max_num_of_ostreams : 30 (../src/amf/amf-sm.c:859)
```

<a id="run_ue"></a>

### Run srsRAN 5G ZMQ UE

Run srsRAN 5G ZMQ UE and connect to Open5GS 5GC.
```
# cd srsRAN_4G/build/srsue
# ./src/srsue ue_zmq.conf
Reading configuration file ue_zmq.conf...

Built in Release mode using commit ec29b0c1f on branch master.

Opening 1 channels in RF device=zmq with args=tx_port=tcp://192.168.0.122:2001,rx_port=tcp://192.168.0.121:2000,base_srate=23.04e6
Supported RF device list: zmq file
CHx base_srate=23.04e6
Current sample rate is 1.92 MHz with a base rate of 23.04 MHz (x12 decimation)
CH0 rx_port=tcp://192.168.0.121:2000
CH0 tx_port=tcp://192.168.0.122:2001
Current sample rate is 23.04 MHz with a base rate of 23.04 MHz (x1 decimation)
Current sample rate is 23.04 MHz with a base rate of 23.04 MHz (x1 decimation)
Waiting PHY to initialize ... done!
Attaching UE...
Random Access Transmission: prach_occasion=0, preamble_index=0, ra-rnti=0x39, tti=334
Random Access Complete.     c-rnti=0x4601, ta=0
RRC Connected
PDU Session Establishment successful. IP: 10.45.0.2
RRC NR reconfiguration successful.
```
The Open5GS C-Plane log when executed is as follows.
```
11/08 23:55:40.204: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:435)
11/08 23:55:40.204: [amf] INFO: [Added] Number of gNB-UEs is now 1 (../src/amf/context.c:2698)
11/08 23:55:40.204: [amf] INFO:     RAN_UE_NGAP_ID[0] AMF_UE_NGAP_ID[1] TAC[1] CellID[0x66c000] (../src/amf/ngap-handler.c:596)
11/08 23:55:40.204: [amf] INFO: [suci-0-001-01-0000-0-0-0000000000] Unknown UE by SUCI (../src/amf/context.c:1865)
11/08 23:55:40.204: [amf] INFO: [Added] Number of AMF-UEs is now 1 (../src/amf/context.c:1641)
11/08 23:55:40.204: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:1224)
11/08 23:55:40.204: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000000]    SUCI (../src/amf/gmm-handler.c:174)
11/08 23:55:40.204: [sbi] INFO: [62b24652-9de1-41ef-afa5-f92528d812a8] NF Instance setup [type:AUSF validity:0s] (../lib/sbi/path.c:297)
11/08 23:55:40.205: [scp] INFO: NF EndPoint(addr) setup [127.0.0.11:7777] (../src/scp/sbi-path.c:429)
11/08 23:55:40.206: [sbi] WARNING: [62b1759c-9de1-41ef-b92f-9dbe62a78ac0] (NRF-discover) NF has already been added [type:UDM] (../lib/sbi/nnrf-handler.c:1249)
11/08 23:55:40.207: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:80] (../lib/sbi/context.c:2199)
11/08 23:55:40.207: [sbi] INFO: NF EndPoint(addr) setup [127.0.0.12:80] (../lib/sbi/context.c:2199)
11/08 23:55:40.207: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.207: [sbi] INFO: NF EndPoint(addr) setup [127.0.0.12:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.207: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.207: [sbi] INFO: NF EndPoint(addr) setup [127.0.0.12:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.207: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.207: [sbi] INFO: NF EndPoint(addr) setup [127.0.0.12:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.207: [sbi] INFO: [62b1759c-9de1-41ef-b92f-9dbe62a78ac0] (NF-discover) NF Profile updated [type:UDM validity:30s] (../lib/sbi/nnrf-handler.c:1290)
11/08 23:55:40.208: [sbi] INFO: [62b49ae2-9de1-41ef-ab2b-a16d704e9f78] NF Instance setup [type:UDR validity:0s] (../lib/sbi/path.c:297)
11/08 23:55:40.208: [scp] INFO: NF EndPoint(addr) setup [127.0.0.20:7777] (../src/scp/sbi-path.c:429)
11/08 23:55:40.210: [sbi] INFO: [UDM] (SCP-discover) NF registered [62b1759c-9de1-41ef-b92f-9dbe62a78ac0] (../lib/sbi/path.c:212)
11/08 23:55:40.210: [sbi] INFO: [62b1759c-9de1-41ef-b92f-9dbe62a78ac0] NF Instance setup [type:UDM validity:0s] (../lib/sbi/path.c:227)
11/08 23:55:40.211: [amf] INFO: NF EndPoint(addr) setup [127.0.0.11:7777] (../src/amf/nausf-handler.c:130)
11/08 23:55:40.249: [scp] INFO: NF EndPoint(addr) setup [127.0.0.11:7777] (../src/scp/sbi-path.c:429)
11/08 23:55:40.250: [scp] INFO: NF EndPoint(addr) setup [127.0.0.20:7777] (../src/scp/sbi-path.c:429)
11/08 23:55:40.252: [ausf] INFO: NF EndPoint(addr) setup [127.0.0.12:7777] (../src/ausf/nudm-handler.c:337)
11/08 23:55:40.290: [sbi] INFO: [62b1759c-9de1-41ef-b92f-9dbe62a78ac0] NF Instance setup [type:UDM validity:0s] (../lib/sbi/path.c:297)
11/08 23:55:40.290: [scp] INFO: NF EndPoint(addr) setup [127.0.0.12:7777] (../src/scp/sbi-path.c:429)
11/08 23:55:40.291: [scp] INFO: NF EndPoint(addr) setup [127.0.0.20:7777] (../src/scp/sbi-path.c:429)
11/08 23:55:40.293: [sbi] INFO: [62b1759c-9de1-41ef-b92f-9dbe62a78ac0] NF Instance setup [type:UDM validity:0s] (../lib/sbi/path.c:297)
11/08 23:55:40.293: [scp] INFO: NF EndPoint(addr) setup [127.0.0.12:7777] (../src/scp/sbi-path.c:429)
11/08 23:55:40.293: [scp] INFO: NF EndPoint(addr) setup [127.0.0.20:7777] (../src/scp/sbi-path.c:429)
11/08 23:55:40.295: [scp] INFO: NF EndPoint(addr) setup [127.0.0.12:7777] (../src/scp/sbi-path.c:429)
11/08 23:55:40.296: [scp] INFO: NF EndPoint(addr) setup [127.0.0.20:7777] (../src/scp/sbi-path.c:429)
11/08 23:55:40.297: [scp] INFO: NF EndPoint(addr) setup [127.0.0.12:7777] (../src/scp/sbi-path.c:429)
11/08 23:55:40.298: [scp] INFO: NF EndPoint(addr) setup [127.0.0.12:7777] (../src/scp/sbi-path.c:429)
11/08 23:55:40.299: [amf] INFO: NF EndPoint(addr) setup [127.0.0.12:7777] (../src/amf/nudm-handler.c:355)
11/08 23:55:40.299: [sbi] INFO: [62b41464-9de1-41ef-9ed7-2d2aa628ee57] NF Instance setup [type:PCF validity:0s] (../lib/sbi/path.c:297)
11/08 23:55:40.299: [scp] INFO: NF EndPoint(addr) setup [127.0.0.13:7777] (../src/scp/sbi-path.c:429)
11/08 23:55:40.300: [pcf] INFO: NF EndPoint(addr) setup [127.0.0.5:7777] (../src/pcf/npcf-handler.c:114)
11/08 23:55:40.301: [sbi] WARNING: [62b49ae2-9de1-41ef-ab2b-a16d704e9f78] (NRF-discover) NF has already been added [type:UDR] (../lib/sbi/nnrf-handler.c:1249)
11/08 23:55:40.301: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:80] (../lib/sbi/context.c:2199)
11/08 23:55:40.301: [sbi] INFO: NF EndPoint(addr) setup [127.0.0.20:80] (../lib/sbi/context.c:2199)
11/08 23:55:40.301: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.301: [sbi] INFO: NF EndPoint(addr) setup [127.0.0.20:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.301: [sbi] INFO: [62b49ae2-9de1-41ef-ab2b-a16d704e9f78] (NF-discover) NF Profile updated [type:UDR validity:30s] (../lib/sbi/nnrf-handler.c:1290)
11/08 23:55:40.302: [sbi] INFO: [UDR] (SCP-discover) NF registered [62b49ae2-9de1-41ef-ab2b-a16d704e9f78] (../lib/sbi/path.c:212)
11/08 23:55:40.302: [sbi] INFO: [62b49ae2-9de1-41ef-ab2b-a16d704e9f78] NF Instance setup [type:UDR validity:0s] (../lib/sbi/path.c:227)
11/08 23:55:40.304: [amf] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../src/amf/npcf-handler.c:143)
11/08 23:55:40.304: [amf] INFO: NF EndPoint(addr) setup [127.0.0.13:7777] (../src/amf/npcf-handler.c:143)
11/08 23:55:40.482: [gmm] INFO: [imsi-001010000000000] Registration complete (../src/amf/gmm-sm.c:2411)
11/08 23:55:40.482: [amf] INFO: [imsi-001010000000000] Configuration update command (../src/amf/nas-path.c:607)
11/08 23:55:40.482: [gmm] INFO:     UTC [2024-11-08T14:55:40] Timezone[0]/DST[0] (../src/amf/gmm-build.c:551)
11/08 23:55:40.482: [gmm] INFO:     LOCAL [2024-11-08T23:55:40] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:556)
11/08 23:55:40.482: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:2719)
11/08 23:55:40.482: [gmm] INFO: UE SUPI[imsi-001010000000000] DNN[internet] S_NSSAI[SST:1 SD:0x1] smContextRef[NULL] smContextResourceURI[NULL] (../src/amf/gmm-handler.c:1368)
11/08 23:55:40.482: [amf] INFO: [62bf8e2a-9de1-41ef-bc1a-256029cbd974] NF Instance setup [type:SMF validity:0s] (../src/amf/context.c:2391)
11/08 23:55:40.482: [gmm] INFO: SMF Instance [62bf8e2a-9de1-41ef-bc1a-256029cbd974] (../src/amf/gmm-handler.c:1409)
11/08 23:55:40.482: [scp] INFO: NF EndPoint(addr) setup [127.0.0.4:7777] (../src/scp/sbi-path.c:429)
11/08 23:55:40.483: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:1031)
11/08 23:55:40.483: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:3119)
11/08 23:55:40.483: [smf] INFO: NF EndPoint(addr) setup [127.0.0.5:7777] (../src/smf/nsmf-handler.c:269)
11/08 23:55:40.484: [sbi] WARNING: [62b1759c-9de1-41ef-b92f-9dbe62a78ac0] (NRF-discover) NF has already been added [type:UDM] (../lib/sbi/nnrf-handler.c:1249)
11/08 23:55:40.484: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:80] (../lib/sbi/context.c:2199)
11/08 23:55:40.484: [sbi] INFO: NF EndPoint(addr) setup [127.0.0.12:80] (../lib/sbi/context.c:2199)
11/08 23:55:40.485: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.485: [sbi] INFO: NF EndPoint(addr) setup [127.0.0.12:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.485: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.485: [sbi] INFO: NF EndPoint(addr) setup [127.0.0.12:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.485: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.485: [sbi] INFO: NF EndPoint(addr) setup [127.0.0.12:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.485: [sbi] INFO: [62b1759c-9de1-41ef-b92f-9dbe62a78ac0] (NF-discover) NF Profile updated [type:UDM validity:30s] (../lib/sbi/nnrf-handler.c:1290)
11/08 23:55:40.485: [scp] INFO: NF EndPoint(addr) setup [127.0.0.20:7777] (../src/scp/sbi-path.c:429)
11/08 23:55:40.488: [sbi] INFO: [UDM] (SCP-discover) NF registered [62b1759c-9de1-41ef-b92f-9dbe62a78ac0] (../lib/sbi/path.c:212)
11/08 23:55:40.488: [sbi] INFO: [62b1759c-9de1-41ef-b92f-9dbe62a78ac0] NF Instance setup [type:UDM validity:0s] (../lib/sbi/path.c:227)
11/08 23:55:40.489: [smf] INFO: NF EndPoint(addr) setup [127.0.0.12:7777] (../src/smf/nudm-handler.c:461)
11/08 23:55:40.490: [amf] INFO: NF EndPoint(addr) setup [127.0.0.4:7777] (../src/amf/nsmf-handler.c:144)
11/08 23:55:40.490: [sbi] WARNING: [62b41464-9de1-41ef-9ed7-2d2aa628ee57] (NRF-discover) NF has already been added [type:PCF] (../lib/sbi/nnrf-handler.c:1249)
11/08 23:55:40.490: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:80] (../lib/sbi/context.c:2199)
11/08 23:55:40.490: [sbi] INFO: NF EndPoint(addr) setup [127.0.0.13:80] (../lib/sbi/context.c:2199)
11/08 23:55:40.490: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.490: [sbi] INFO: NF EndPoint(addr) setup [127.0.0.13:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.491: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.491: [sbi] INFO: NF EndPoint(addr) setup [127.0.0.13:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.491: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.491: [sbi] INFO: NF EndPoint(addr) setup [127.0.0.13:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.491: [sbi] INFO: [62b41464-9de1-41ef-9ed7-2d2aa628ee57] (NF-discover) NF Profile updated [type:PCF validity:30s] (../lib/sbi/nnrf-handler.c:1290)
11/08 23:55:40.491: [pcf] INFO: NF EndPoint(addr) setup [127.0.0.4:7777] (../src/pcf/npcf-handler.c:447)
11/08 23:55:40.492: [sbi] WARNING: [62b49ae2-9de1-41ef-ab2b-a16d704e9f78] (NRF-discover) NF has already been added [type:UDR] (../lib/sbi/nnrf-handler.c:1249)
11/08 23:55:40.492: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:80] (../lib/sbi/context.c:2199)
11/08 23:55:40.492: [sbi] INFO: NF EndPoint(addr) setup [127.0.0.20:80] (../lib/sbi/context.c:2199)
11/08 23:55:40.492: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.492: [sbi] INFO: NF EndPoint(addr) setup [127.0.0.20:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.492: [sbi] INFO: [62b49ae2-9de1-41ef-ab2b-a16d704e9f78] (NF-discover) NF Profile updated [type:UDR validity:30s] (../lib/sbi/nnrf-handler.c:1290)
11/08 23:55:40.494: [sbi] WARNING: [UDR] (SCP-discover) NF has already been added [62b49ae2-9de1-41ef-ab2b-a16d704e9f78] (../lib/sbi/path.c:217)
11/08 23:55:40.494: [sbi] INFO: [62b49ae2-9de1-41ef-ab2b-a16d704e9f78] NF Instance setup [type:UDR validity:0s] (../lib/sbi/path.c:227)
11/08 23:55:40.494: [pcf] INFO: [62b41464-9de1-41ef-9ed7-2d2aa628ee57] NF Instance setup [type:PCF validity:0s] (../src/pcf/nudr-handler.c:227)
11/08 23:55:40.495: [sbi] WARNING: [62b1931a-9de1-41ef-b76e-31aab7a0ca0b] (NRF-discover) NF has already been added [type:BSF] (../lib/sbi/nnrf-handler.c:1249)
11/08 23:55:40.495: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.15:80] (../lib/sbi/context.c:2199)
11/08 23:55:40.495: [sbi] INFO: NF EndPoint(addr) setup [127.0.0.15:80] (../lib/sbi/context.c:2199)
11/08 23:55:40.495: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.15:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.495: [sbi] INFO: NF EndPoint(addr) setup [127.0.0.15:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.495: [sbi] INFO: [62b1931a-9de1-41ef-b76e-31aab7a0ca0b] (NF-discover) NF Profile updated [type:BSF validity:30s] (../lib/sbi/nnrf-handler.c:1290)
11/08 23:55:40.496: [sbi] INFO: [BSF] (SCP-discover) NF registered [62b1931a-9de1-41ef-b76e-31aab7a0ca0b] (../lib/sbi/path.c:212)
11/08 23:55:40.496: [sbi] INFO: [62b1931a-9de1-41ef-b76e-31aab7a0ca0b] NF Instance setup [type:BSF validity:0s] (../lib/sbi/path.c:227)
11/08 23:55:40.496: [pcf] INFO: NF EndPoint(addr) setup [127.0.0.15:7777] (../src/pcf/nbsf-handler.c:151)
11/08 23:55:40.498: [sbi] INFO: [PCF] (SCP-discover) NF registered [62b41464-9de1-41ef-9ed7-2d2aa628ee57] (../lib/sbi/path.c:212)
11/08 23:55:40.498: [sbi] INFO: [62b41464-9de1-41ef-9ed7-2d2aa628ee57] NF Instance setup [type:PCF validity:0s] (../lib/sbi/path.c:227)
11/08 23:55:40.498: [smf] INFO: NF EndPoint(addr) setup [127.0.0.13:7777] (../src/smf/npcf-handler.c:367)
11/08 23:55:40.498: [smf] INFO: UE SUPI[imsi-001010000000000] DNN[internet] IPv4[10.45.0.2] IPv6[] (../src/smf/npcf-handler.c:578)
11/08 23:55:40.499: [gtp] INFO: gtp_connect() [192.168.0.112]:2152 (../lib/gtp/path.c:60)
11/08 23:55:40.529: [scp] INFO: NF EndPoint(addr) setup [127.0.0.4:7777] (../src/scp/sbi-path.c:429)
11/08 23:55:40.531: [sbi] WARNING: [62b1759c-9de1-41ef-b92f-9dbe62a78ac0] (NRF-discover) NF has already been added [type:UDM] (../lib/sbi/nnrf-handler.c:1249)
11/08 23:55:40.531: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:80] (../lib/sbi/context.c:2199)
11/08 23:55:40.531: [sbi] INFO: NF EndPoint(addr) setup [127.0.0.12:80] (../lib/sbi/context.c:2199)
11/08 23:55:40.531: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.531: [sbi] INFO: NF EndPoint(addr) setup [127.0.0.12:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.531: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.531: [sbi] INFO: NF EndPoint(addr) setup [127.0.0.12:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.531: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.531: [sbi] INFO: NF EndPoint(addr) setup [127.0.0.12:7777] (../lib/sbi/context.c:1938)
11/08 23:55:40.531: [sbi] INFO: [62b1759c-9de1-41ef-b92f-9dbe62a78ac0] (NF-discover) NF Profile updated [type:UDM validity:30s] (../lib/sbi/nnrf-handler.c:1290)
11/08 23:55:40.531: [sbi] INFO: [62b49ae2-9de1-41ef-ab2b-a16d704e9f78] NF Instance setup [type:UDR validity:0s] (../lib/sbi/path.c:297)
11/08 23:55:40.532: [scp] INFO: NF EndPoint(addr) setup [127.0.0.20:7777] (../src/scp/sbi-path.c:429)
11/08 23:55:40.533: [sbi] WARNING: [UDM] (SCP-discover) NF has already been added [62b1759c-9de1-41ef-b92f-9dbe62a78ac0] (../lib/sbi/path.c:217)
11/08 23:55:40.533: [sbi] INFO: [62b1759c-9de1-41ef-b92f-9dbe62a78ac0] NF Instance setup [type:UDM validity:0s] (../lib/sbi/path.c:227)
11/08 23:55:40.533: [amf] INFO: [imsi-001010000000000:1:11][0:0:NULL] /nsmf-pdusession/v1/sm-contexts/{smContextRef}/modify (../src/amf/nsmf-handler.c:920)
11/08 23:55:40.650: [gmm] INFO: [imsi-001010000000000] No GUTI allocated (../src/amf/gmm-sm.c:1572)
```
The Open5GS U-Plane log when executed is as follows.
```
11/08 23:55:40.517: [upf] INFO: [Added] Number of UPF-Sessions is now 1 (../src/upf/context.c:209)
11/08 23:55:40.517: [gtp] INFO: gtp_connect() [192.168.0.111]:2152 (../lib/gtp/path.c:60)
11/08 23:55:40.517: [upf] INFO: UE F-SEID[UP:0x5a8 CP:0xd7e] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:495)
11/08 23:55:40.517: [upf] INFO: UE F-SEID[UP:0x5a8 CP:0xd7e] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:495)
11/08 23:55:40.549: [gtp] INFO: gtp_connect() [192.168.0.121]:2152 (../lib/gtp/path.c:60)
```
The result of `ip addr show` on VM4 (UE) is as follows.
```
# ip addr show
...
6: tun_srsue: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.45.0.2/24 scope global tun_srsue
       valid_lft forever preferred_lft forever
...
```

<a id="ping"></a>

## Ping google.com

Specify the TUN interface on VM4 (UE) and try `ping`.

<a id="ping_1"></a>

### Case for going through DN 10.45.0.0/16

Execute `tcpdump` on VM2 (U-Plane) and check that the packet goes through `if=ogstun`.
- `ping google.com` on VM4 (UE)
```
# ping google.com -I tun_srsue -n
PING google.com (172.217.31.174) from 10.45.0.2 tun_srsue: 56(84) bytes of data.
64 bytes from 172.217.31.174: icmp_seq=1 ttl=111 time=43.1 ms
64 bytes from 172.217.31.174: icmp_seq=2 ttl=111 time=47.5 ms
64 bytes from 172.217.31.174: icmp_seq=3 ttl=111 time=52.8 ms
```
- Run `tcpdump` on VM2 (U-Plane)
```
# tcpdump -i ogstun -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on ogstun, link-type RAW (Raw IP), snapshot length 262144 bytes
23:58:49.595631 IP 10.45.0.2 > 172.217.31.174: ICMP echo request, id 3, seq 1, length 64
23:58:49.614006 IP 172.217.31.174 > 10.45.0.2: ICMP echo reply, id 3, seq 1, length 64
23:58:50.602918 IP 10.45.0.2 > 172.217.31.174: ICMP echo request, id 3, seq 2, length 64
23:58:50.619890 IP 172.217.31.174 > 10.45.0.2: ICMP echo reply, id 3, seq 2, length 64
23:58:51.608848 IP 10.45.0.2 > 172.217.31.174: ICMP echo request, id 3, seq 3, length 64
23:58:51.626154 IP 172.217.31.174 > 10.45.0.2: ICMP echo reply, id 3, seq 3, length 64
```
In addition to `ping`, you may try to access the web by specifying the TUNnel interface with `curl` as follows.
- `curl google.com` on VM4 (UE)
```
# curl --interface tun_srsue google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```
- Run `tcpdump` on VM2 (U-Plane)
```
23:59:29.896115 IP 10.45.0.2.51864 > 172.217.31.174.80: Flags [S], seq 2397604747, win 64240, options [mss 1460,sackOK,TS val 1995507748 ecr 0,nop,wscale 7], length 0
23:59:29.913411 IP 172.217.31.174.80 > 10.45.0.2.51864: Flags [S.], seq 3568963808, ack 2397604748, win 65535, options [mss 1412,sackOK,TS val 1964248535 ecr 1995507748,nop,wscale 8], length 0
23:59:29.939783 IP 10.45.0.2.51864 > 172.217.31.174.80: Flags [.], ack 1, win 502, options [nop,nop,TS val 1995507796 ecr 1964248535], length 0
23:59:29.939819 IP 10.45.0.2.51864 > 172.217.31.174.80: Flags [P.], seq 1:75, ack 1, win 502, options [nop,nop,TS val 1995507796 ecr 1964248535], length 74: HTTP: GET / HTTP/1.1
23:59:29.958775 IP 172.217.31.174.80 > 10.45.0.2.51864: Flags [.], ack 75, win 1050, options [nop,nop,TS val 1964248581 ecr 1995507796], length 0
23:59:29.995304 IP 172.217.31.174.80 > 10.45.0.2.51864: Flags [P.], seq 1:774, ack 75, win 1050, options [nop,nop,TS val 1964248617 ecr 1995507796], length 773: HTTP: HTTP/1.1 301 Moved Permanently
23:59:30.025533 IP 10.45.0.2.51864 > 172.217.31.174.80: Flags [.], ack 774, win 501, options [nop,nop,TS val 1995507877 ecr 1964248617], length 0
23:59:30.025571 IP 10.45.0.2.51864 > 172.217.31.174.80: Flags [F.], seq 75, ack 774, win 501, options [nop,nop,TS val 1995507878 ecr 1964248617], length 0
23:59:30.042475 IP 172.217.31.174.80 > 10.45.0.2.51864: Flags [F.], seq 774, ack 76, win 1050, options [nop,nop,TS val 1964248664 ecr 1995507878], length 0
23:59:30.055547 IP 10.45.0.2.51864 > 172.217.31.174.80: Flags [.], ack 775, win 501, options [nop,nop,TS val 1995507924 ecr 1964248664], length 0
```
You could now create the end-to-end TUN interface on the DN and send any packets on the network.

---
In investigating 5G SA, I have built a simulation environment and can now use a very useful system for investigating 5GC and MEC of 5G SA mobile network. I would like to thank the excellent developers and all the contributors of Open5GS, srsRAN Project and srsRAN 4G.

<a id="changelog"></a>

## Changelog (summary)

- [2024.11.08] Updated to Open5GS v2.7.2 (2024.10.18), and gnb_zmq.yaml according to srsRAN_Project 24.10 (2024.10.14).
- [2024.03.31] [This commit](https://github.com/open5gs/open5gs/commit/e8a3b76af395a9986234b7d339a7a96dc5bb537f) fixed the issue where SMF crashes without `gtpc` section in `smf.yaml`. So deleted the `gtpc` section in `smf.yaml` for 5G use.
- [2024.03.29] Updated to Open5GS v2.7.0 (2024.03.24).
- [2023.11.02] Updated `gnb_zmq.yaml`.
- [2023.10.21] Updated `gnb_zmq.yaml` according to srsRAN_Project 23.10 (2023.10.20).
- [2023.10.11] Updated Open5GS v2.6.6 (2023.10.10) and srsRAN_Project (2023.09.20).
- [2023.08.26] Initial release.
