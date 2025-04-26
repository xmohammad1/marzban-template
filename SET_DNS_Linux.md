# Setting DNS on Ubuntu 22.04
```
cat /etc/resolv.conf && sudo rm -f /etc/resolv.conf && echo -e "nameserver 1.1.1.1\nnameserver 8.8.8.8" | sudo tee /etc/resolv.conf > /dev/null && cat /etc/resolv.conf && sudo systemctl stop systemd-resolved && sudo systemctl disable systemd-resolved
```

This guide demonstrates how to configure Google DNS servers on Ubuntu 22.04 using both Netplan and systemd-resolved.

## Method 1: Using Netplan

Netplan is the network configuration utility in Ubuntu that uses YAML files to configure network interfaces.

### Step 1: Identify your network configuration file

Netplan configuration files are located in `/etc/netplan/`. First, identify your config file:

```bash
ls /etc/netplan/
```

### Step 2: Edit the Netplan configuration file

Open the configuration file with a text editor (replace with your actual filename):

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

### Step 3: Add Google DNS servers

Modify the file to include the nameservers. Here's an example configuration:

```yaml
network:
    version: 2
    ethernets:
        eth0:
            addresses:
            - 2a01:4f9:c010:1148::1/64
            dhcp4: true
            dhcp4-overrides:
                use-dns: false
            match:
                macaddress: 96:00:04:1e:be:7e
            nameservers:
                addresses:
                    - 1.1.1.1
                    - 8.8.8.8
```

### Step 4: Apply the configuration

Apply the changes with:

```bash
sudo netplan apply
```

### Step 5: Verify DNS servers

Check if the DNS servers are configured correctly:

```bash
resolvectl status
```

## Method 2: Using systemd-resolved directly

You can also configure DNS settings through systemd-resolved's configuration file.

### Step 1: Edit the systemd-resolved configuration

Open the systemd-resolved configuration file:

```bash
sudo nano /etc/systemd/resolved.conf
```

### Step 2: Set Google DNS servers

Uncomment or add the DNS line and add Google's DNS servers:

```
[Resolve]
DNS=1.1.1.1
FallbackDNS=8.8.8.8
```

### Step 3: Restart systemd-resolved

Apply changes by restarting the service:

```bash
sudo systemctl restart systemd-resolved
```

### Step 4: Verify the configuration

Check if the DNS settings are applied:

```bash
resolvectl status
```

## Troubleshooting

If DNS resolution isn't working as expected:

1. Ensure the configuration files are correct and properly formatted.
2. Verify that systemd-resolved is running:
   ```bash
   systemctl status systemd-resolved
   ```
3. Check the symlink for `/etc/resolv.conf`:

   ```bash
   ls -l /etc/resolv.conf
   ```
   to point
   ```ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf```

## Additional Notes
if DNS is not fully set you can search which files contain dns
```
grep -r "2.56.220.2" /etc/
```
- If your system uses a VPN or other network managers, they might override these settings.
- For a consistent configuration, you may want to disable other DNS resolvers that might conflict with systemd-resolved.

## References

- [Netplan Configuration Examples](https://netplan.io/examples/)
- [systemd-resolved Documentation](https://www.freedesktop.org/software/systemd/man/resolved.conf.html)
- [Google Public DNS](https://developers.google.com/speed/public-dns/)
