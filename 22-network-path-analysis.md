# Lab 22 - Network Path Analysis and Packet Capture Tools

## Objective

Master network path diagnostic utilities and packet inspection tools on Linux. By completing this lab, you will learn how to trace network hops with `traceroute` and `mtr`, capture raw network packets using `tcpdump` and `tshark`, construct capture filters, export traffic to `.pcap` files, and analyze traffic graphically using Wireshark.

By completing this lab, I should be able to:

* Trace Layer 3 network paths hop-by-hop using `traceroute` and interpret `* * *` timeout responses
* Perform real-time continuous latency and packet loss diagnostic analysis using `mtr` (My Traceroute)
* Capture live network packets using `tcpdump` across specific interfaces (`-i`), save to PCAP files (`-w`), and apply host/source/destination primitive filters
* Understand network infrastructure mirroring techniques (Port Mirroring / SPAN)
* Leverage CLI and GUI packet analyzers (`tshark`, `wireshark`) for in-depth protocol inspection

---

## Part 1 - Network Path Analysis (`traceroute` & `mtr`)

### Path Diagnostic Tools Comparison

| Feature / Tool | `traceroute` | `mtr` (My Traceroute) |
| :--- | :--- | :--- |
| **Primary Function** | Static hop-by-hop path tracing | Dynamic, continuous path analysis |
| **Key Output Metrics** | Round-trip time (RTT) for individual hops | Real-time packet loss %, latency, jitter, and statistics |
| **Timeout Symbol** | `* * *` (indicates ICMP block, firewall drop, or rate-limiting) | Displayed as 100% loss at specific hop |
| **Protocol Switches** | Default: UDP/ICMP | Default: ICMP (`-u` for UDP, `-t` for TCP) |

---

## Part 2 - Packet Capture & Filtering (`tcpdump`, `tshark`, `wireshark`)

Packet capture tools allow low-level inspection of network traffic passing through network interfaces.

### 1. `tcpdump` Flag Reference

| Flag | Description | Syntax Example |
| :--- | :--- | :--- |
| **`-i <iface>`** | Select specific capture interface | `tcpdump -i eth0` |
| **`-w <file>`** | Write raw packet payload output to a `.pcap` file | `tcpdump -w capture.pcap` |
| **`-r <file>`** | Read and display packets from a `.pcap` file | `tcpdump -r capture.pcap` |
| **`-n`** | Disable reverse DNS lookup for IP addresses | `tcpdump -n` |

### 2. Primitive Capture Filters

Capture filters restrict which packets are parsed and logged to memory or disk.

* **Filter by Host (Either Source or Destination):**
  ```bash
  tcpdump host 192.168.1.1
  ```
* **Filter by Source Host Only:**
  ```bash
  tcpdump src host 192.168.1.50
  ```
* **Filter by Destination Host Only:**
  ```bash
  tcpdump dst host 8.8.8.8
  ```
* **Filter by Port and Protocol:**
  ```bash
  tcpdump tcp port 80 or port 443
  ```

### 3. TShark and Wireshark

* **Wireshark:** Premier GUI-based packet capture and deep-packet analysis suite.
* **TShark:** Terminal-based version of Wireshark. Uses the **`-f`** option to specify capture filters (Berkeley Packet Filters).
* **Port Mirroring / SPAN (Switched Port Analyzer):** Managed network switch feature that duplicates network traffic from one or more switch ports and sends it to a monitoring port connected to a packet capture appliance or analyzer.

---

## Part 3 - Hands-On Practice Lab

### Step 1: Trace Network Paths with `traceroute` and `mtr`

1. Run a standard traceroute to a public target:
   ```bash
   traceroute 8.8.8.8
   ```
   *(Observe any hops displaying `* * *`, which indicate firewalls blocking ICMP responses).*

2. Launch interactive path analysis using `mtr`:
   ```bash
   mtr 8.8.8.8
   ```
3. Test path diagnostics using TCP mode in `mtr`:
   ```bash
   mtr --tcp -P 80 8.8.8.8
   ```

### Step 2: Live Packet Capture with `tcpdump`

1. List available network interfaces for capture:
   ```bash
   sudo tcpdump -D
   ```
2. Capture 10 ICMP/ping packets on interface `eth0` (or `any`) and save them to a PCAP file:
   ```bash
   sudo tcpdump -i any -c 10 -w /tmp/ping_test.pcap icmp
   ```
3. Open a second terminal window and generate ping traffic:
   ```bash
   ping -c 4 1.1.1.1
   ```
4. Read and inspect the captured PCAP file using `tcpdump`:
   ```bash
   sudo tcpdump -n -r /tmp/ping_test.pcap
   ```

### Step 3: Targeted Host Filtering

1. Capture traffic originating specifically from your local system IP to an external host:
   ```bash
   sudo tcpdump -i any dst host 8.8.8.8
   ```

---

## Completion Checklist

- [x] I understand that `* * *` in `traceroute` indicates a hop timed out or blocked ICMP
- [x] I know that `mtr` combines traceroute and ping metrics, offering `-u` (UDP) and `-t` (TCP) modes
- [x] I can select capture interfaces with `tcpdump -i` and export captures to PCAP files using `-w`
- [x] I can construct directional host filters (`host`, `src host`, `dst host`)
- [x] I know that TShark uses `-f` to define capture filters
- [x] I understand that Port Mirroring / SPAN duplicates network switch traffic to a packet analyzer port
