# Open5GS 5GC & srsRAN 5G with ZeroMQ UE / RAN Sample Configuration
srsRAN_Project and srsRAN_4G software suites include a virtual radio which uses the ZeroMQ networking library to transfer radio samples between applications.
Therefore, in order to use U-Plane's DN (Data Network) as a trial, I built a simulation environment for the 5GC mobile network.
This briefly describes the overall and configuration files in the Virtualbox VM environment.

---

<a id="conf_list"></a>

## List of Sample Configurations

1. [One SGW-C/PGW-C, one SGW-U/PGW-U and one APN](https://github.com/s5uishida/open5gs_epc_srsran_sample_config)
2. [One SGW-C/PGW-C, Multiple SGW-Us/PGW-Us and APNs](https://github.com/s5uishida/open5gs_epc_oai_sample_config)
3. One SMF, one UPF and one DNN (this article)
4. [One SMF, Multiple UPFs and DNNs](https://github.com/s5uishida/open5gs_5gc_ueransim_sample_config)
5. [Select nearby UPF(PGW-U) according to the connected eNodeB](https://github.com/s5uishida/open5gs_epc_srsran_nearby_upf_sample_config)
6. [Select nearby UPF according to the connected gNodeB](https://github.com/s5uishida/open5gs_5gc_ueransim_nearby_upf_sample_config)
7. [Select UPF based on S-NSSAI](https://github.com/s5uishida/open5gs_5gc_ueransim_snssai_upf_sample_config)
8. [SCP Indirect communication Model C](https://github.com/s5uishida/open5gs_5gc_ueransim_scp_model_c_sample_config)
9. [VoLTE and SMS Configuration for docker_open5gs](https://github.com/s5uishida/docker_open5gs_volte_sms_config)
10. [Monitoring Metrics with Prometheus](https://github.com/s5uishida/open5gs_5gc_ueransim_metrics_sample_config)
11. [Framed Routing](https://github.com/s5uishida/open5gs_5gc_ueransim_framed_routing_sample_config)
12. [VPP-UPF(PGW-U) with DPDK](https://github.com/s5uishida/open5gs_epc_srsran_vpp_upf_dpdk_sample_config)
13. [VPP-UPF with DPDK](https://github.com/s5uishida/open5gs_5gc_ueransim_vpp_upf_dpdk_sample_config)
---

<a id="misc"></a>

## Miscellaneous Notes

- [Install MongoDB 6.0 and Open5GS WebUI](https://github.com/s5uishida/open5gs_install_mongodb6_webui)
- [Install MongoDB 4.4.18 on Ubuntu 20.04 for Raspberry Pi 4B](https://github.com/s5uishida/install_mongodb_on_ubuntu_for_rp4b)
- [Build srsRAN_Project 5G RAN with ZeroMQ](https://github.com/s5uishida/build_srsran_5g_zmq)
- [Build srsRAN 4G UE / RAN with ZeroMQ by disabling RF plugins](https://github.com/s5uishida/build_srsran_4g_zmq_disable_rf_plugins)
- [A Note for 5G SUCI Profile A/B Scheme](https://github.com/s5uishida/note_5g_suci_profile_ab)
- [A Note for Changing Network Interface of UPF from TUN to TAP in Open5GS](https://github.com/s5uishida/change_from_tun_to_tap_in_open5gs)
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
- 5GC - Open5GS v2.6.6 (2023.10.10) - https://github.com/open5gs/open5gs
- RAN - srsRAN Project (2023.09.20) - https://github.com/srsran/srsRAN_Project
- UE (NR-UE) - srsRAN 4G (2023.06.19) - https://github.com/srsran/srsRAN_4G

Each VMs are as follows.  
| VM | SW & Role | IP address | OS | CPU <br> (Min) | Memory <br> (Min) | HDD <br> (Min) |
| --- | --- | --- | --- | --- | --- | --- |
| VM1 | Open5GS 5GC C-Plane | 192.168.0.111/24 | Ubuntu 22.04 | 1 | 1GB | 20GB |
| VM2 | Open5GS 5GC U-Plane  | 192.168.0.112/24 | Ubuntu 22.04 | 1 | 1GB | 20GB |
| VM3 | srsRAN Project ZMQ RAN<br>(gNodeB) | 192.168.0.121/24 | Ubuntu 22.04 | 2 | 4GB | 10GB |
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
- Open5GS v2.6.6 (2023.10.10) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- srsRAN Project (RAN) (2023.09.20) - https://github.com/s5uishida/build_srsran_5g_zmq
- srsRAN 4G (UE) (2023.06.19) - https://github.com/s5uishida/build_srsran_4g_zmq_disable_rf_plugins

<a id="changes_cp"></a>

### Changes in configuration files of Open5GS 5GC C-Plane

The following parameters including DNN can be used in the logic that selects UPF as the connection destination by PFCP.

- DNN
- TAC (Tracking Area Code)
- nr_CellID

For the sake of simplicity, I used only DNN this time. Please refer to [here](https://github.com/open5gs/open5gs/pull/560#issue-483001043) for the logic to select UPF.

<a id="changes_cp"></a>

### Changes in configuration files of Open5GS 5GC C-Plane

- `open5gs/install/etc/open5gs/amf.yaml`
```diff
--- amf.yaml.orig       2023-08-26 17:34:58.000000000 +0900
+++ amf.yaml    2023-08-26 17:55:14.356847505 +0900
@@ -474,28 +474,29 @@
       - addr: 127.0.0.5
         port: 7777
     ngap:
-      - addr: 127.0.0.5
+      - addr: 192.168.0.111
     metrics:
       - addr: 127.0.0.5
         port: 9090
     guami:
       - plmn_id:
-          mcc: 999
-          mnc: 70
+          mcc: 001
+          mnc: 01
         amf_id:
           region: 2
           set: 1
     tai:
       - plmn_id:
-          mcc: 999
-          mnc: 70
+          mcc: 001
+          mnc: 01
         tac: 1
     plmn_support:
       - plmn_id:
-          mcc: 999
-          mnc: 70
+          mcc: 001
+          mnc: 01
         s_nssai:
           - sst: 1
+            sd: 000001
     security:
         integrity_order : [ NIA2, NIA1, NIA0 ]
         ciphering_order : [ NEA0, NEA1, NEA2 ]
```
- `open5gs/install/etc/open5gs/smf.yaml`
```diff
--- smf.yaml.orig       2023-08-26 17:34:58.000000000 +0900
+++ smf.yaml    2023-08-26 17:54:41.609843715 +0900
@@ -602,25 +602,20 @@
       - addr: 127.0.0.4
         port: 7777
     pfcp:
-      - addr: 127.0.0.4
-      - addr: ::1
+      - addr: 192.168.0.111
     gtpc:
       - addr: 127.0.0.4
-      - addr: ::1
     gtpu:
-      - addr: 127.0.0.4
-      - addr: ::1
+      - addr: 192.168.0.111
     metrics:
       - addr: 127.0.0.4
         port: 9090
     subnet:
       - addr: 10.45.0.1/16
-      - addr: 2001:db8:cafe::1/48
+        dnn: internet
     dns:
       - 8.8.8.8
       - 8.8.4.4
-      - 2001:4860:4860::8888
-      - 2001:4860:4860::8844
     mtu: 1400
     ctf:
       enabled: auto
@@ -808,7 +803,8 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.112
+        dnn: internet
 
 #
 #  o Disable use of IPv4 addresses (only IPv6)
```

<a id="changes_up"></a>

### Changes in configuration files of Open5GS 5GC U-Plane

- `open5gs/install/etc/open5gs/upf.yaml`
```diff
--- upf.yaml.orig       2023-08-26 17:34:58.000000000 +0900
+++ upf.yaml    2023-08-26 17:57:55.068477294 +0900
@@ -196,12 +196,13 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.112
     gtpu:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.112
     subnet:
       - addr: 10.45.0.1/16
-      - addr: 2001:db8:cafe::1/48
+        dnn: internet
+        dev: ogstun
     metrics:
       - addr: 127.0.0.7
         port: 9090
```

<a id="changes_srs"></a>

### Changes in configuration files of srsRAN 5G ZMQ UE / RAN

<a id="changes_ran"></a>

#### Changes in configuration files of RAN

See [here](https://github.com/s5uishida/build_srsran_5g_zmq#create-the-configuration-file-of-gnodeb) for the original files.

- `srsRAN_Project/build/apps/gnb/gnb_zmq.yaml`
```diff
--- gnb_zmq.yaml.orig   2023-07-07 00:32:21.000000000 +0900
+++ gnb_zmq.yaml        2023-08-26 18:06:39.534791708 +0900
@@ -3,13 +3,19 @@
 # To run the srsRAN Project gNB with this config, use the following command: 
 #   sudo ./gnb -c gnb_zmq.yaml
 
+gnb_id: 0x19B
+
+slicing:
+  - sst: 1
+    sd: 1
+
 amf:
-  addr: 10.53.1.2                  # The address or hostname of the AMF.
-  bind_addr: 10.53.1.1             # A local IP that the gNB binds to for traffic from the AMF.
+  addr: 192.168.0.111                  # The address or hostname of the AMF.
+  bind_addr: 192.168.0.121             # A local IP that the gNB binds to for traffic from the AMF.
 
 ru_sdr:
   device_driver: zmq                # The RF driver name.
-  device_args: tx_port=tcp://127.0.0.1:2000,rx_port=tcp://127.0.0.1:2001,base_srate=11.52e6 # Optionally pass arguments to the selected RF driver.
+  device_args: tx_port=tcp://192.168.0.121:2000,rx_port=tcp://192.168.0.122:2001,base_srate=11.52e6 # Optionally pass arguments to the selected RF driver.
   srate: 11.52                      # RF sample rate might need to be adjusted according to selected bandwidth.
   tx_gain: 75                       # Transmit gain of the RF might need to adjusted to the given situation.
   rx_gain: 75                       # Receive gain of the RF might need to adjusted to the given situation.
@@ -20,7 +26,7 @@
   channel_bandwidth_MHz: 10         # Bandwith in MHz. Number of PRBs will be automatically derived.
   common_scs: 15                    # Subcarrier spacing in kHz used for data.
   plmn: "00101"                     # PLMN broadcasted by the gNB.
-  tac: 7                            # Tracking area code (needs to match the core configuration).
+  tac: 1                            # Tracking area code (needs to match the core configuration).
   pdcch:
     dedicated:
       ss2_type: common              # Search Space type, has to be set to common
```

<a id="changes_ue"></a>

#### Changes in configuration files of UE

See [here](https://github.com/s5uishida/build_srsran_4g_zmq_disable_rf_plugins#create-the-configuration-file-of-nr-ue) for the original files.

- `srsRAN_4G/build/srsue/ue_zmq.conf`
```diff
--- ue_zmq.conf.orig    2023-04-24 18:36:33.000000000 +0900
+++ ue_zmq.conf 2023-08-26 18:09:17.043564563 +0900
@@ -6,7 +6,7 @@
 nof_antennas = 1
 
 device_name = zmq
-device_args = tx_port=tcp://127.0.0.1:2001,rx_port=tcp://127.0.0.1:2000,base_srate=11.52e6
+device_args = tx_port=tcp://192.168.0.122:2001,rx_port=tcp://192.168.0.121:2000,base_srate=11.52e6
 
 [rat.eutra]
 dl_earfcn = 2850
@@ -32,9 +32,9 @@
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
@@ -42,11 +42,16 @@
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
- Open5GS v2.6.6 (2023.10.10) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- srsRAN Project (RAN) (2023.09.20) - https://github.com/s5uishida/build_srsran_5g_zmq
- srsRAN 4G (UE) (2023.06.19) - https://github.com/s5uishida/build_srsran_4g_zmq_disable_rf_plugins

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
Lower PHY in executor blocking mode.

--== srsRAN gNB (commit 5e6f50a20) ==--

Connecting to AMF on 192.168.0.111:38412
Available radio types: zmq.
Cell pci=1, bw=10 MHz, dl_arfcn=368500 (n3), dl_freq=1842.5 MHz, dl_ssb_arfcn=368410, ul_freq=1747.5 MHz

==== gNodeB started ===
Type <t> to view trace
```
The Open5GS C-Plane log when executed is as follows.
```
10/11 20:34:57.910: [amf] INFO: gNB-N2 accepted[192.168.0.121]:48052 in ng-path module (../src/amf/ngap-sctp.c:113)
10/11 20:34:57.910: [amf] INFO: gNB-N2 accepted[192.168.0.121] in master_sm module (../src/amf/amf-sm.c:741)
10/11 20:34:57.910: [amf] INFO: [Added] Number of gNBs is now 1 (../src/amf/context.c:1185)
10/11 20:34:57.910: [amf] INFO: gNB-N2[192.168.0.121] max_num_of_ostreams : 30 (../src/amf/amf-sm.c:780)
```

<a id="run_ue"></a>

### Run srsRAN 5G ZMQ UE

Run srsRAN 5G ZMQ UE and connect to Open5GS 5GC.
```
# cd srsRAN_4G/build/srsue
# ./src/srsue ue_zmq.conf
Reading configuration file ue_zmq.conf...

Built in Release mode using commit fa56836b1 on branch master.

Opening 1 channels in RF device=zmq with args=tx_port=tcp://192.168.0.122:2001,rx_port=tcp://192.168.0.121:2000,base_srate=11.52e6
Supported RF device list: zmq file
CHx base_srate=11.52e6
Current sample rate is 1.92 MHz with a base rate of 11.52 MHz (x6 decimation)
CH0 rx_port=tcp://192.168.0.121:2000
CH0 tx_port=tcp://192.168.0.122:2001
Current sample rate is 11.52 MHz with a base rate of 11.52 MHz (x1 decimation)
Current sample rate is 11.52 MHz with a base rate of 11.52 MHz (x1 decimation)
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
10/11 20:35:48.444: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:401)
10/11 20:35:48.444: [amf] INFO: [Added] Number of gNB-UEs is now 1 (../src/amf/context.c:2523)
10/11 20:35:48.444: [amf] INFO:     RAN_UE_NGAP_ID[0] AMF_UE_NGAP_ID[1] TAC[1] CellID[0x19b0] (../src/amf/ngap-handler.c:562)
10/11 20:35:48.444: [amf] INFO: [suci-0-001-01-0000-0-0-0000000000] Unknown UE by SUCI (../src/amf/context.c:1789)
10/11 20:35:48.444: [amf] INFO: [Added] Number of AMF-UEs is now 1 (../src/amf/context.c:1570)
10/11 20:35:48.444: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:1061)
10/11 20:35:48.444: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000000]    SUCI (../src/amf/gmm-handler.c:157)
10/11 20:35:48.447: [sbi] WARNING: [2e5782e8-682a-41ee-a705-499077b49eb3] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:833)
10/11 20:35:48.447: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1623)
10/11 20:35:48.447: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1532)
10/11 20:35:48.447: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1532)
10/11 20:35:48.447: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1532)
10/11 20:35:48.448: [sbi] INFO: [2e5782e8-682a-41ee-a705-499077b49eb3] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:856)
10/11 20:35:48.598: [sbi] WARNING: [2e5e0190-682a-41ee-8e27-abb8e1c20f56] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:833)
10/11 20:35:48.598: [sbi] WARNING: NF EndPoint updated [127.0.0.20:80] (../lib/sbi/context.c:1623)
10/11 20:35:48.598: [sbi] WARNING: NF EndPoint updated [127.0.0.20:7777] (../lib/sbi/context.c:1532)
10/11 20:35:48.598: [sbi] INFO: [2e5e0190-682a-41ee-8e27-abb8e1c20f56] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:856)
10/11 20:35:48.784: [gmm] INFO: [imsi-001010000000000] Registration complete (../src/amf/gmm-sm.c:1993)
10/11 20:35:48.784: [amf] INFO: [imsi-001010000000000] Configuration update command (../src/amf/nas-path.c:612)
10/11 20:35:48.784: [gmm] INFO:     UTC [2023-10-11T11:35:48] Timezone[0]/DST[0] (../src/amf/gmm-build.c:558)
10/11 20:35:48.784: [gmm] INFO:     LOCAL [2023-10-11T20:35:48] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:563)
10/11 20:35:48.785: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:2544)
10/11 20:35:48.785: [gmm] INFO: UE SUPI[imsi-001010000000000] DNN[internet] S_NSSAI[SST:1 SD:0x1] (../src/amf/gmm-handler.c:1247)
10/11 20:35:48.786: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:1010)
10/11 20:35:48.786: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:3057)
10/11 20:35:48.788: [sbi] WARNING: [2e5782e8-682a-41ee-a705-499077b49eb3] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:833)
10/11 20:35:48.788: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1623)
10/11 20:35:48.788: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1532)
10/11 20:35:48.788: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1532)
10/11 20:35:48.788: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1532)
10/11 20:35:48.788: [sbi] INFO: [2e5782e8-682a-41ee-a705-499077b49eb3] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:856)
10/11 20:35:48.792: [sbi] WARNING: [2e5e2e22-682a-41ee-89c7-c55c10685709] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:833)
10/11 20:35:48.793: [sbi] WARNING: NF EndPoint updated [127.0.0.13:80] (../lib/sbi/context.c:1623)
10/11 20:35:48.793: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1532)
10/11 20:35:48.793: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1532)
10/11 20:35:48.793: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1532)
10/11 20:35:48.793: [sbi] INFO: [2e5e2e22-682a-41ee-89c7-c55c10685709] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:856)
10/11 20:35:48.795: [sbi] WARNING: [2e5e0190-682a-41ee-8e27-abb8e1c20f56] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:833)
10/11 20:35:48.795: [sbi] WARNING: NF EndPoint updated [127.0.0.20:80] (../lib/sbi/context.c:1623)
10/11 20:35:48.795: [sbi] WARNING: NF EndPoint updated [127.0.0.20:7777] (../lib/sbi/context.c:1532)
10/11 20:35:48.796: [sbi] INFO: [2e5e0190-682a-41ee-8e27-abb8e1c20f56] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:856)
10/11 20:35:48.798: [sbi] WARNING: [2e55269c-682a-41ee-a9d4-b7f30fd1a910] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:833)
10/11 20:35:48.798: [sbi] WARNING: NF EndPoint updated [127.0.0.15:80] (../lib/sbi/context.c:1623)
10/11 20:35:48.798: [sbi] WARNING: NF EndPoint updated [127.0.0.15:7777] (../lib/sbi/context.c:1532)
10/11 20:35:48.798: [sbi] INFO: [2e55269c-682a-41ee-a9d4-b7f30fd1a910] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:856)
10/11 20:35:48.801: [smf] INFO: UE SUPI[imsi-001010000000000] DNN[internet] IPv4[10.45.0.2] IPv6[] (../src/smf/npcf-handler.c:539)
10/11 20:35:48.802: [gtp] INFO: gtp_connect() [192.168.0.112]:2152 (../lib/gtp/path.c:60)
10/11 20:35:48.825: [gmm] INFO: [imsi-001010000000000] No GUTI allocated (../src/amf/gmm-sm.c:1323)
10/11 20:35:48.954: [sbi] WARNING: [2e5782e8-682a-41ee-a705-499077b49eb3] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:833)
10/11 20:35:48.954: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1623)
10/11 20:35:48.954: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1532)
10/11 20:35:48.954: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1532)
10/11 20:35:48.954: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1532)
10/11 20:35:48.955: [sbi] INFO: [2e5782e8-682a-41ee-a705-499077b49eb3] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:856)
10/11 20:35:48.957: [amf] INFO: [imsi-001010000000000:1:11][0:0:NULL] /nsmf-pdusession/v1/sm-contexts/{smContextRef}/modify (../src/amf/nsmf-handler.c:837)
```
The Open5GS U-Plane log when executed is as follows.
```
10/11 20:35:48.791: [upf] INFO: [Added] Number of UPF-Sessions is now 1 (../src/upf/context.c:206)
10/11 20:35:48.791: [gtp] INFO: gtp_connect() [192.168.0.111]:2152 (../lib/gtp/path.c:60)
10/11 20:35:48.791: [upf] INFO: UE F-SEID[UP:0x3c9 CP:0x7c9] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:483)
10/11 20:35:48.791: [upf] INFO: UE F-SEID[UP:0x3c9 CP:0x7c9] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:483)
10/11 20:35:48.942: [gtp] INFO: gtp_connect() [192.168.0.121]:2152 (../lib/gtp/path.c:60)
```
The result of `ip addr show` on VM4 (UE) is as follows.
```
# ip addr show
...
5: tun_srsue: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 500
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
PING google.com (142.251.222.14) from 10.45.0.2 tun_srsue: 56(84) bytes of data.
64 bytes from 142.251.222.14: icmp_seq=1 ttl=61 time=99.1 ms
64 bytes from 142.251.222.14: icmp_seq=2 ttl=61 time=71.1 ms
64 bytes from 142.251.222.14: icmp_seq=3 ttl=61 time=71.9 ms
```
- Run `tcpdump` on VM2 (U-Plane)
```
# tcpdump -i ogstun -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on ogstun, link-type RAW (Raw IP), snapshot length 262144 bytes
20:38:08.302516 IP 10.45.0.2 > 142.251.222.14: ICMP echo request, id 4, seq 1, length 64
20:38:08.343237 IP 142.251.222.14 > 10.45.0.2: ICMP echo reply, id 4, seq 1, length 64
20:38:09.298801 IP 10.45.0.2 > 142.251.222.14: ICMP echo request, id 4, seq 2, length 64
20:38:09.315035 IP 142.251.222.14 > 10.45.0.2: ICMP echo reply, id 4, seq 2, length 64
20:38:10.297658 IP 10.45.0.2 > 142.251.222.14: ICMP echo request, id 4, seq 3, length 64
20:38:10.312919 IP 142.251.222.14 > 10.45.0.2: ICMP echo reply, id 4, seq 3, length 64
```
In addition to `ping`, you may try to access the web by specifying the TUNnel interface with `curl` as follows.
- Run `curl google.com` on VM4 (UE)
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
20:38:57.723802 IP 10.45.0.2.45166 > 142.251.222.14.80: Flags [S], seq 2746546244, win 64240, options [mss 1460,sackOK,TS val 3051146979 ecr 0,nop,wscale 7], length 0
20:38:57.739849 IP 142.251.222.14.80 > 10.45.0.2.45166: Flags [S.], seq 2176001, ack 2746546245, win 65535, options [mss 1460], length 0
20:38:57.789814 IP 10.45.0.2.45166 > 142.251.222.14.80: Flags [.], ack 1, win 64240, length 0
20:38:57.790039 IP 10.45.0.2.45166 > 142.251.222.14.80: Flags [P.], seq 1:75, ack 1, win 64240, length 74: HTTP: GET / HTTP/1.1
20:38:57.790157 IP 142.251.222.14.80 > 10.45.0.2.45166: Flags [.], ack 75, win 65535, length 0
20:38:57.851657 IP 142.251.222.14.80 > 10.45.0.2.45166: Flags [P.], seq 1:774, ack 75, win 65535, length 773: HTTP: HTTP/1.1 301 Moved Permanently
20:38:57.885404 IP 10.45.0.2.45166 > 142.251.222.14.80: Flags [.], ack 774, win 63467, length 0
20:38:57.885452 IP 10.45.0.2.45166 > 142.251.222.14.80: Flags [F.], seq 75, ack 774, win 63467, length 0
20:38:57.885548 IP 142.251.222.14.80 > 10.45.0.2.45166: Flags [.], ack 76, win 65535, length 0
20:38:57.908163 IP 142.251.222.14.80 > 10.45.0.2.45166: Flags [F.], seq 774, ack 76, win 65535, length 0
20:38:57.951607 IP 10.45.0.2.45166 > 142.251.222.14.80: Flags [.], ack 775, win 63467, length 0
```
You could now create the end-to-end TUN interface on the DN and send any packets on the network.

---
In investigating 5G SA, I have built a simulation environment and can now use a very useful system for investigating 5GC and MEC of 5G SA mobile network. I would like to thank the excellent developers and all the contributors of Open5GS, srsRAN Project and srsRAN 4G.

<a id="changelog"></a>

## Changelog (summary)

- [2023.10.11] Updated Open5GS v2.6.6 (2023.10.10) and srsRAN_Project (2023.09.20).
- [2023.08.26] Initial release.
