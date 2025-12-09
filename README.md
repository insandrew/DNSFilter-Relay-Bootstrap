# DNSFilter Relay Bootstrap

A streamlined Bash utility designed to automate the deployment and configuration of [DNSFilter Relays](https://help.dnsfilter.com/hc/en-us/articles/1500008107562-DNS-Relay-Deployment-Guide) on Ubuntu 22.04 virtual machines.

This script transforms the manual, multi-step process of setting up a Relay into a single interactive session, handling everything from OS hardening to persistent network configuration.

## 🚀 Features

* **Interactive Setup:** Prompts for Hostname, Static IP, Gateway, and DNSFilter API Keys once, then runs unattended.
* **System Hardening:**
    * Configures `systemd-resolved` to free up Port 53.
    * Sets up UFW firewall rules (allowing DNS and SSH).
    * Enables Unattended Upgrades for security patches.
* **Service Management:** Downloads the Relay binary, generates the config, and installs it as a generic systemd service.
* **Network Persistence:**
    * Generates a clean Netplan configuration.
    * Creates `cloud-init` overrides to ensure the Static IP persists across reboots.

## 📋 Prerequisites

* **OS:** Ubuntu 22.04 LTS (Fresh Install recommended).
* **Hardware:** 1 vCPU, 1GB RAM, 20GB Disk.
* **Network:** Access to the internet to download binaries and updates.
* **DNSFilter Data:** You will need your **Site Secret Key** from the DNSFilter dashboard.

## 🛠️ Usage

1.  **Provision your VM:** Install Ubuntu 22.04 and log in via SSH (using the DHCP IP).
2.  **Clone or Download the script:**

    ```bash
    # Clone the repo
    git clone https://github.com/insandrew/dnsfilter-relay-bootstrap.git
    cd dnsfilter-relay-bootstrap
    
    # Make executable
    chmod +x setup_relay.sh
    ```

3.  **Run the script:**
    
    ```bash
    sudo ./setup_relay.sh
    ```

4.  **Follow the prompts:**
    You will be asked to provide:
    * Desired Hostname
    * New Static IP / Gateway / DNS
    * DNSFilter Secret Key & Site Name
    * Internal Domain list (for local resolution routing)

### ⚠️ Important Note on Connectivity
The script applies network changes as the **final step**. 
If you are changing the IP address from DHCP to Static, **your SSH session will disconnect** when the script finishes. You must reconnect using the new Static IP defined during the setup.

## 🔧 What the Script Does (Under the Hood)

1.  **Input Collection:** Gathers all necessary network and API variables.
2.  **System Prep:** Sets the hostname, updates `apt` repositories, and installs dependencies (`unzip`, `wget`, `cron`).
3.  **Port 53 Liberation:** Modifies `/etc/systemd/resolved.conf` to disable the stub listener, allowing the Relay to bind to port 53.
4.  **Installation:** Downloads the latest `relay-linux-amd64` binary to `/bin/relay/`.
5.  **Configuration:** Generates a `relay.conf` file based on your inputs (mapping local domains to your local DNS servers).
6.  **Service Creation:** Creates and enables `/lib/systemd/system/relay.service`.
7.  **Network Switch:** Wipes existing Netplan configs, writes a new YAML file for the static IP, and disables Cloud-Init network configuration to prevent overwrites.

## 🧪 Validation

After the script completes and you have reconnected via SSH:

1.  **Check Service Status:**
    ```bash
    sudo systemctl status relay.service
    ```
2.  **Test External Resolution:**
    ```bash
    nslookup google.com <RELAY_IP>
    ```
3.  **Test Internal Resolution:**
    ```bash
    nslookup internal-server.corp.local <RELAY_IP>
    ```

## 📜 License

This project is licensed under the MIT License - see the LICENSE file for details.

---
*Maintained by [INS Andrew](https://github.com/insandrew)*
