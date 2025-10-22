# Fix Virt-Manager / Libvirt NAT networking on Arch Linux

## 1. Install dependencies

```bash
sudo pacman -S --needed qemu-desktop virt-manager libvirt dnsmasq iptables-nft dmidecode edk2-ovmf
```

## 2. Enable and start libvirt service
```bash
sudo systemctl enable --now libvirtd
```
## 3. Make sure nftables backend is used
```bash
sudo ln -sf /usr/bin/iptables-nft /usr/bin/iptables
sudo ln -sf /usr/bin/ip6tables-nft /usr/bin/ip6tables
```
## 4. Load required kernel modules

``` bash
sudo modprobe bridge
sudo modprobe tun
sudo modprobe vhost_net
```
## Make them persistent
``` bash
echo -e "bridge\ntun\nvhost_net" | sudo tee /etc/modules-load.d/libvirt.conf
```

## 5. Add current user to the libvirt group (relogin after this)


``` bash
sudo usermod -aG libvirt $USER
```
## 6. Recreate the default NAT network (if missing or broken)
```bash
cat << 'EOF' | sudo tee /tmp/default.xml
<network>
  <name>default</name>
  <forward mode='nat'/>
  <bridge name='virbr0' stp='on' delay='0'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
    </dhcp>
  </ip>
</network>
EOF

```
```bash
sudo systemctl restart libvirtd
sudo virsh net-define /tmp/default.xml
sudo virsh net-autostart default
sudo virsh net-start default
```
## 7. Verify network status

```bash
sudo virsh net-list --all

```
### Expected output:

```bash
Name      State    Autostart
--------------------------------
default   active   yes
```
