# Detailed Technical Write-Up: Multi-Interface Setup & Troubleshooting for Cisco CSR1000v in VMware Workstation

---

## 1. Executive Summary & Objective

The objective of this task was to expand a single-interface **Cisco CSR1000v** virtual router into a multi-interface topology capable of supporting a multi-subnet automation lab. 

The target architecture requires dedicated network segments for:
1. **Management & Automation (`GigabitEthernet1`)**: Dedicated host-only connection reserved for Ansible orchestration and SSH management (`192.168.229.0/24`).
2. **Simulated LAN Segment 1 (`GigabitEthernet2`)**: Isolated host-only network (`192.168.10.0/24`).
3. **Simulated LAN Segment 2 (`GigabitEthernet3`)**: Isolated host-only network (`192.168.20.0/24`).

---

## 2. Network Topology & Subnet Design

| Router Interface | VMware Adapter | Virtual Switch | Network Type | Subnet / Role | DHCP Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **GigabitEthernet1** | Network Adapter 1 | `VMnet1` | Host-only | `192.168.229.0/24` (Management / Ansible) | Enabled |
| **GigabitEthernet2** | Network Adapter 2 | `VMnet2` | Custom (Host-only) | `192.168.10.0/24` (LAN 1) | Disabled |
| **GigabitEthernet3** | Network Adapter 3 | `VMnet3` | Custom (Host-only) | `192.168.20.0/24` (LAN 2) | Disabled |

---

## 3. Step-by-Step Configuration Workflows

### Step 3.1: Virtual Network Editor Setup (VMware Workstation)
To isolate traffic between subnets without interfering with management or external interfaces:
1. Navigated to **Edit $
ightarrow$ Virtual Network Editor** in VMware Workstation and unlocked administrative privileges via **Change Settings**.
2. Created two new custom Host-only virtual networks: **VMnet2** and **VMnet3**.
3. **Disabled Local DHCP Services**: Unchecked `Use local DHCP service to distribute IP address to VMs` on both `VMnet2` and `VMnet3` to ensure full control over static IP addressing via Cisco IOS-XE and Ansible.
4. Set Subnet IPs:
   * **VMnet2**: `192.168.10.0` (Netmask: `255.255.255.0`)
   * **VMnet3**: `192.168.20.0` (Netmask: `255.255.255.0`)

### Step 3.2: Virtual Machine Hardware Association
1. Powered off the `CSR1000v` Virtual Machine.
2. Under **Virtual Machine Settings $
ightarrow$ Hardware**:
   * Set **Network Adapter 2** to `Custom: Specific virtual network` $
ightarrow$ **`VMnet2`**.
   * Added **Network Adapter 3** and set to `Custom: Specific virtual network` $
ightarrow$ **`VMnet3`**.
   * Ensured `Connect at power on` was checked for all adapters.

---

## 4. Problem Encountered & Root Cause Analysis

### The Issue
Upon powering on the virtual machine and executing `show ip interface brief` in the Cisco IOS-XE CLI, **only `GigabitEthernet1` was detected**:

```text
CSR-Nahj# show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       192.168.229.129 YES DHCP   up                    up
```

`GigabitEthernet2` and `GigabitEthernet3` were completely absent from the operating system's interface list despite being present in VMware's VM hardware settings.

### Root Cause Identification
Inspecting the VM's underlying `.vmx` configuration file revealed the driver mismatch:

```text
ethernet1.virtualDev = "e1000"
ethernet2.virtualDev = "e1000"
```

1. **Virtual Network Driver Incompatibility**: By default, VMware Workstation attached new network interfaces using the legacy Intel `e1000` driver. Cisco IOS-XE on the CSR1000v requires VMware's **`vmxnet3`** paravirtualized network driver to properly bind hardware controllers to the OS kernel.
2. **Lack of Hot-Plug Support**: Adding NICs while the router was running or suspended prevented the IOS-XE Linux kernel from enumerating PCI bus changes.

---

## 5. Troubleshooting & Resolution

### Step 5.1: `.vmx` File Modification
1. Gracefully shut down the router and completely closed VMware Workstation to clear file locks (`.vmx.lck`).
2. Opened `CSR1000v_for_VMware.vmx` in Notepad.
3. Modified the `virtualDev` parameter for `ethernet1` and `ethernet2` from `e1000` to **`vmxnet3`**:

```text
ethernet1.connectionType = "custom"
ethernet1.addressType = "generated"
ethernet1.virtualDev = "vmxnet3"
ethernet2.connectionType = "custom"
ethernet2.addressType = "generated"
ethernet2.virtualDev = "vmxnet3"
```

4. Saved the changes to `CSR1000v_for_VMware.vmx`.

### Step 5.2: Hardware Enumeration Boot
1. Launched VMware Workstation and initiated a cold boot of the CSR1000v router.
2. During initialization, the Cisco IOS-XE kernel scanned the virtual PCI bus, detected the two new `vmxnet3` adapters, and assigned them as `GigabitEthernet2` and `GigabitEthernet3`.

---

## 6. Final Verification & Outcome

After the system boot completed, running `show ip interface brief` confirmed that all three physical virtual interfaces were active and recognized by Cisco IOS-XE:

```text
CSR-Nahj# show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       192.168.229.129 YES DHCP   up                    up
GigabitEthernet2       unassigned      YES unset  administratively down down
GigabitEthernet3       unassigned      YES unset  administratively down down
```

