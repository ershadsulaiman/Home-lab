# Enterprise Home Lab Infrastructure

**Overview**: This repository documents the architecture and configuration of a custom-built, bare-metal home lab environment engineered for enterprise systems administration, network security research, and penetration testing. It provides a segmented, highly available infrastructure utilizing Proxmox virtualization, centralized Windows identity management, and encrypted remote access overlays for realistic corporate network simulation.

## Key Features
* **Hypervisor Virtualization**: Powered by a Proxmox VE hypervisor running on an Intel i5-7500 with 32GB RAM, hosting multiple server and client virtual machines with biweekly automated snapshot routines.
* **Centralized Identity & Access**: Fully configured Windows Server 2025 Active Directory domain controller managing DNS, DHCP, Organizational Units, and group policies for domain-joined Windows 11 clients.
* **Network Segmentation & Routing**: Utilizes an OPNsense firewall gateway routing traffic through a physical 24-port managed switch (2960-24TT) to enforce strict subnet isolation.
* **VLAN Isolation**: Traffic is segmented into dedicated logical networks, explicitly separating infrastructure compute resources (VLAN 801) from control endpoints (VLAN 901).
* **Secure Remote Administration**: Implements Tailscale on a dedicated Raspberry Pi SBC node to provide encrypted, zero-trust remote SSH and SPICE access to internal lab resources without exposing public ports.
* **Security Operations Workstation**: Maintains a dedicated Kali Linux virtual machine provisioned with Nmap, Ghidra, and reverse engineering tools for continuous vulnerability scanning and CTF training against internal lab assets.
* **Advanced Layer 2 Topologies**: Leverages VXLAN overlays to seamlessly bridge Layer 2 broadcast domains across disparate, segmented subnets.

## Tech Stack & Architecture
* **Virtualization**: Proxmox VE
* **Identity & OS**: Windows Server 2025, Windows 11, Kali Linux
* **Networking & Security**: OPNsense, Tailscale (WireGuard VPN), Cisco IOS
* **Hardware**: Intel Core i5 Desktop, Raspberry Pi SBC

```text
home-lab/
├── docs/
│   ├── active-directory-setup.md
│   ├── network-topology.md
│   └── proxmox-deployment.md
├── configs/
│   ├── opnsense/
│   │   └── backup-config.xml
│   └── switch/
│       └── 2960-24TT-running-config.cfg
├── scripts/
│   ├── ad-user-provisioning.ps1
│   ├── snapshot-vms.sh
│   └── tailscale-deploy.sh
└── README.md
```

## Getting Started

### Prerequisites
* Proxmox VE 8.x installed on bare-metal hardware (minimum 32GB RAM recommended).
* Windows Server 2025 and Windows 11 ISOs uploaded to the Proxmox storage pool.
* OPNsense firewall and a Layer 2 managed switch (e.g., Cisco 2960 series).
* A Tailscale account and an active pre-authorized authentication key.

### Installation
1. Clone this repository to your local administration machine:
   ```bash
   git clone [https://github.com/ershaaad/home-lab.git](https://github.com/ershaaad/home-lab.git)
   cd home-lab
   ```

2. Apply the base managed switch configuration to establish your VLANs:
   ```bash
   # Connect via serial console or initial SSH
   cat configs/switch/2960-24TT-running-config.cfg | ssh admin@10.0.0.2
   ```

3. Provision the Raspberry Pi remote access node:
   ```bash
   cd scripts
   chmod +x tailscale-deploy.sh
   ./tailscale-deploy.sh
   ```

### Environment Variables
For automated script execution (such as Proxmox API interactions or automated Tailscale deployment), create a `.env` file in the root directory:

```env
# .env.example
PROXMOX_API_URL=[https://10.0.80.10:8006/api2/json](https://10.0.80.10:8006/api2/json)
PROXMOX_API_TOKEN_ID=root@pam!automation
PROXMOX_API_TOKEN_SECRET=your_api_secret_here
TAILSCALE_AUTH_KEY=tskey-auth-your_key_here
AD_DOMAIN_NAME=lab.local
```

## Usage Examples

**Automated Active Directory User Creation**
To rapidly populate the Windows Server 2025 environment with test users, utilize the provided PowerShell provisioning script on the domain controller:

```powershell
# Run as Administrator on Windows Server 2025
.\scripts\ad-user-provisioning.ps1 -Domain "lab.local" -UserCount 50 -TargetOU "OU=TestUsers,DC=lab,DC=local"
```

**Internal Subnet Reconnaissance**
To verify network segmentation and discover exposed services from the Kali Linux VM:

```bash
# Execute from the Kali workstation terminal
sudo nmap -sV -p- -T4 10.0.80.0/24 -oN scans/vlan801-baseline.txt
```

## Roadmap & Contributing
- [ ] Implement a high-availability (HA) cluster configuration for Proxmox VE nodes.
- [ ] Deploy Security Information and Event Management (SIEM) software (e.g., Splunk or Wazuh) for centralized log aggregation.
- [ ] Automate OPNsense firewall rule backups to this repository via CI/CD pipelines.

**Contributing**
Pull requests are welcome. For major changes or topology alterations, please open an issue first to discuss what you would like to change. Ensure all configuration scripts are stripped of sensitive API keys, passwords, and private IP ranges before committing.

## License
This project is licensed under the MIT License.
