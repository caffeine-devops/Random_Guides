# Configure Static IP address on Ubuntu server using Netplan

This guide walks you through setting a **static IP address** on an Ubuntu server using **Netplan**, for both **Wi-Fi** and **Ethernet** interfaces.

> Note:
> This should also work for other Linux servers/systems

---

## Prerequisites

- Running Linux system (in my case Ubuntu server 22.04 LTS)
- `sudo` privileges
- Your network interface name (e.g., `wlp2s0` for Wi-Fi, `ens18` or `enp3s0` for Ethernet)
- IP addressing details (desired static IP, gateway, DNS)

---

## Step 1: Backup existing Netplan configuration

Always backup the original config before making changes:

```bash
# for knowing the config file name (which is a yaml file)
ls /etc/netplan/

# backup the config file
sudo cp /etc/netplan/<filename> /etc/netplan/<filename.bkp>
```

## Step 2: Edit the Netplan YAML File

Open the Netplan config file:

```bash
sudo vi /etc/netplan/<filename>
# if you used vi like me, you can save the file and quit by pressing <esc> and then :wq
```

Now modify and use the below example files as per your usecase.

### Example: Static IP for Ethernet Interface

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens18:                              # Update to your actual Ethernet interface name
      addresses:
        - 192.168.XXX.XXX/24               # Replace with your desired static IP
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
      routes:
        - to: default
          via: 192.168.XXX.XXX              # Replace with your router’s IP
```

### Example: Static IP for Wi-Fi Interface

```yaml
network:
  version: 2
  renderer: networkd
  wifis:
    wlp2s0:                             # Update to your actual Wi-Fi interface name
      addresses:
        - 192.168.XXX.XXX/24              # Replace with your desired static IP
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
      routes:
        - to: default
          via: 192.168.XXX.XXX              # Replace with your router’s IP
      access-points:
        "my_wifi_name":                # Replace with your SSID
          auth:
            key-management: "psk"
            password: "your_wifi_password"      # Replace with your wifi password
```

> Note: Be sure to update the interface names (`wlp2s0`,`ens18`, etc) to match your system. Use the `ip a` or `ifconfig` or `networkctl` command to list the interface.
> Also update any relevant details that I have highlighted with the comments

## Step 3: Apply the Configuration

Save the file and apply the changes:

```bash
sudo netplan apply
```

For troubleshooting and detailed logs:

```bash
sudo netplan --debug apply
```

## Step 4: Verify the Configuration

Check IP address:

```bash
ip a show wlp2s0     # for Wi-Fi
ip a show ens18      # for Ethernet
```

Check gateway:

```bash
ip route
```

Ping to verify connectivity:

```bash
ping www.google.com -c4
```

## Step 5: Rollback if Needed

If the network breaks or you lose connectivity:

1. Connect via recovery mode or direct access (if remote)
2. Restore backup:

    ```bash
    sudo cp /etc/netplan/<filename.bkp> /etc/netplan/<filename>
    sudo netplan apply
    ```

---

## Tips

- Use `ip a` to find your active interface name
- Avoid assigning static IPs inside your router's DHCP range
- `networkctl` shows interface statuses
