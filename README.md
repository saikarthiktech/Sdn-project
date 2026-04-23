# 🚀 Simple QoS Priority Controller using SDN

## 📌 Project Overview

This project implements a **Quality of Service (QoS) Priority Controller** using **Software Defined Networking (SDN)**.  
The controller classifies network traffic and assigns different priority levels based on traffic type.

---

## 🎯 Objective

- Identify different types of network traffic  
- Assign priority levels (LOW, MEDIUM, HIGH)  
- Install flow rules using SDN controller  
- Observe and analyze traffic behavior  

---

## 🧠 Concept

### 🔹 QoS (Quality of Service)
QoS ensures that important traffic gets higher priority over less important traffic.

### 🔹 SDN (Software Defined Networking)
SDN separates the control plane from the data plane:
- Controller (POX) → decision making  
- Switch (Mininet) → forwarding  

---

## ⚙️ Technologies Used

- Python  
- POX Controller  
- Mininet  
- Ubuntu (WSL)  

---

## 🏗️ Network Topology

- 1 Switch  
- 3 Hosts (h1, h2, h3)  
- Remote Controller (POX)  

---

## 🔄 Working Flow

1. Host sends packet  
2. Switch forwards packet to controller  
3. Controller analyzes packet  
4. Assigns priority based on protocol  
5. Installs flow rule  
6. Packet is forwarded  

---

## 📊 Priority Mapping

| Traffic Type | Protocol | Priority |
|-------------|--------|----------|
| Ping        | ICMP   | LOW      |
| HTTP        | TCP:80 | MEDIUM   |
| UDP (iperf) | UDP    | HIGH     |

---

## 💻 Controller Code (qos_controller.py)
```python
from pox.core import core
import pox.openflow.libopenflow_01 as of

log = core.getLogger()

def _handle_PacketIn(event):
    try:
        packet = event.parsed
        if not packet:
            return

        priority = "LOW"

        tcp = packet.find('tcp')
        udp = packet.find('udp')

        if udp:
            priority = "HIGH"
        elif tcp:
            if tcp.dstport == 80:
                priority = "MEDIUM"

        log.info("Packet priority: %s", priority)

        msg = of.ofp_packet_out()
        msg.data = event.ofp
        msg.actions.append(of.ofp_action_output(port=of.OFPP_FLOOD))
        event.connection.send(msg)

    except Exception as e:
        log.error("Error: %s", e)

def launch():
    core.openflow.addListenerByName("PacketIn", _handle_PacketIn)
    log.info("QoS Controller Started")

## 🚀 To Run the Project

### Step 1: Start Controller
cd ~/pox  
./pox.py log.level --INFO openflow.of_01 forwarding.l2_learning qos_controller  

### Step 2: Start Mininet
wsl -u root  
mn --topo single,3 --controller remote  

### Step 3: Test Network
pingall  

### Step 4: Generate Traffic

LOW Priority:
h1 ping -c 2 h2  

MEDIUM Priority:
h1 python3 -m http.server 80 &  
h2 wget http://10.0.0.1  

HIGH Priority:
h1 iperf -s -u &  
h2 iperf -c 10.0.0.1 -u  
