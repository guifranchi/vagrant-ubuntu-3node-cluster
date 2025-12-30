# JBoss EAP Multi-Cluster Lab Environment

A portable Vagrant-based lab environment for testing JBoss EAP clustering and High Availability (HA) configurations on Apple Silicon Macs [web:44].

## Overview

This project provides a reproducible, three-node cluster environment specifically designed for JBoss EAP development, testing, and learning. Each node runs Ubuntu 24.04 LTS with OpenJDK 17 pre-installed and is configured for seamless inter-node communication.

## Architecture

- **3 Virtual Machines**: Ubuntu 24.04 LTS (ARM64)
- **Java**: OpenJDK 17 (JBoss EAP compatible)
- **Networking**: 
  - Private network for cluster communication (192.168.56.0/24)
  - NAT network for internet access
  - Port forwarding for service access from host
- **Hostname Resolution**: Configured `/etc/hosts` for node discovery

### Node Configuration

| Node | Hostname | IP Address | Memory | CPUs |
|------|----------|------------|--------|------|
| node1 | jboss-node1 | 192.168.56.11 | 2048 MB | 2 |
| node2 | jboss-node2 | 192.168.56.12 | 2048 MB | 2 |
| node3 | jboss-node3 | 192.168.56.13 | 2048 MB | 2 |

## Prerequisites

- **macOS** with Apple Silicon (M1/M2/M3/M4)
- **VMware Fusion** (Free for personal use) [web:44]
- **Vagrant** 2.4.7 or later
- **Vagrant VMware Utility** [web:49]
- **vagrant-vmware-desktop plugin**

### Installation Steps

#### 1. Install VMware Fusion Pro

Download VMware Fusion Pro (free for personal use) from Broadcom [web:44].

#### 2. Install Vagrant

```
brew install hashicorp/tap/hashicorp-vagrant
```

#### 3. Install Vagrant VMware Utility

```
brew install --cask vagrant-vmware-utility
```

#### 4. Install VMware Provider Plugin

```
vagrant plugin install vagrant-vmware-desktop
```

#### 5. Configure Vagrant VMware Utility

```
# Generate certificates
sudo /opt/vagrant-vmware-desktop/bin/vagrant-vmware-utility certificate generate

# Install and start service
sudo /opt/vagrant-vmware-desktop/bin/vagrant-vmware-utility service install

# Verify service is running
sudo launchctl list | grep vagrant-vmware-utility
```

## Quick Start

```
# Clone the repository
git clone <your-repo-url>
cd jboss-cluster-lab

# Start all nodes
vagrant up --provider=vmware_desktop

# Verify all nodes are running
vagrant status

# SSH into any node
vagrant ssh node1
```

## Port Mappings

Access services from your Mac using localhost and the following ports:

### Node1 (jboss-node1)

| Service | Guest Port | Host Port |
|---------|------------|-----------|
| SSH | 22 | 2201 |
| HTTP | 80 | 8001 |
| HTTPS | 443 | 8431 |
| Custom HTTP | 8080 | 8081 |
| Custom HTTPS | 8443 | 8444 |
| Management (8990) | 8990 | 8991 |
| Management (9990) | 9990 | 9991 |

### Node2 (jboss-node2)

| Service | Guest Port | Host Port |
|---------|------------|-----------|
| SSH | 22 | 2202 |
| HTTP | 80 | 8002 |
| HTTPS | 443 | 8432 |
| Custom HTTP | 8080 | 8082 |
| Custom HTTPS | 8443 | 8445 |
| Management (8990) | 8990 | 8992 |
| Management (9990) | 9990 | 9992 |

### Node3 (jboss-node3)

| Service | Guest Port | Host Port |
|---------|------------|-----------|
| SSH | 22 | 2203 |
| HTTP | 80 | 8003 |
| HTTPS | 443 | 8433 |
| Custom HTTP | 8080 | 8083 |
| Custom HTTPS | 8443 | 8446 |
| Management (8990) | 8990 | 8993 |
| Management (9990) | 9990 | 9993 |

## Common Commands

```
# Start all nodes
vagrant up --provider=vmware_desktop

# Start specific node
vagrant up node1 --provider=vmware_desktop

# SSH into nodes
vagrant ssh node1
vagrant ssh node2
vagrant ssh node3

# Check status of all nodes
vagrant status

# Stop all nodes
vagrant halt

# Stop specific node
vagrant halt node2

# Restart nodes
vagrant reload

# Restart with reprovisioning (reinstall Java, update /etc/hosts)
vagrant reload --provision

# Destroy all nodes
vagrant destroy -f

# Destroy specific node
vagrant destroy node3 -f
```

## Network Testing

Verify inter-node communication:

```
# SSH into node1
vagrant ssh node1

# Test connectivity to other nodes
ping -c 2 jboss-node2
ping -c 2 jboss-node3

# Verify hostname resolution
cat /etc/hosts

# Check Java installation
java -version
echo $JAVA_HOME
```

## JBoss EAP Setup

### Installing JBoss EAP

Once your nodes are running, you can install JBoss EAP on each node [web:16]:

```
# SSH into each node
vagrant ssh node1

# Download JBoss EAP (requires Red Hat subscription)
# Or use WildFly (open-source version)
wget https://github.com/wildfly/wildfly/releases/download/XX.X.X/wildfly-XX.X.X.tar.gz

# Extract
tar xzf wildfly-XX.X.X.tar.gz
sudo mv wildfly-XX.X.X /opt/wildfly

# Set ownership
sudo chown -R vagrant:vagrant /opt/wildfly
```

### Clustering Configuration

For JBoss EAP clustering in standalone-ha mode, use these bind addresses [web:16]:

```
# Example startup for node1
cd /opt/wildfly/bin
./standalone.sh -c standalone-ha.xml \
  -b 0.0.0.0 \
  -bmanagement 0.0.0.0 \
  -bprivate 192.168.56.11 \
  -Djboss.node.name=node1 \
  -Djboss.default.multicast.address=230.0.0.5
```

**Key Parameters:**
- `-b 0.0.0.0` - Public interface (accessible from all networks)
- `-bmanagement 0.0.0.0` - Management interface
- `-bprivate 192.168.56.X` - Cluster communication interface (use each node's private IP)
- `-Djboss.node.name` - Unique node identifier
- `-Djboss.default.multicast.address` - Multicast address for cluster discovery

### Accessing Management Console

From your Mac, access each node's management console:

- **Node1**: http://localhost:9991
- **Node2**: http://localhost:9992
- **Node3**: http://localhost:9993

## Customization

### Adjusting Resources

Edit the `Vagrantfile` to change memory or CPU allocation:

```
config.vm.provider "vmware_desktop" do |vmware|
  vmware.memory = "4096"  # Increase to 4GB
  vmware.cpus = 4         # Increase to 4 CPUs
end
```

Then reload:
```
vagrant reload
```

### Adding More Nodes

To add a fourth node, add this block to the `Vagrantfile`:

```
config.vm.define "node4" do |node4|
  node4.vm.hostname = "jboss-node4"
  node4.vm.network "private_network", ip: "192.168.56.14"
  
  node4.vm.network "forwarded_port", guest: 22, host: 2204
  node4.vm.network "forwarded_port", guest: 9990, host: 9994
  # ... add other ports
  
  node4.vm.provider "vmware_desktop" do |vmware|
    vmware.vmx["displayname"] = "jboss-node4"
  end
  
  node4.vm.provision "shell", inline: $install_java
  node4.vm.provision "shell", inline: $configure_hosts
end
```

Don't forget to update the `$configure_hosts` script to include the new node.

## Troubleshooting

### VMs Won't Start

```
# Reset VMware networking
sudo /Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --stop
sudo /Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --start
```

### Certificate Errors

```
# Regenerate Vagrant VMware Utility certificates
sudo /opt/vagrant-vmware-desktop/bin/vagrant-vmware-utility certificate generate
sudo launchctl unload /Library/LaunchDaemons/com.vagrant.vagrant-vmware-utility.plist
sudo launchctl load /Library/LaunchDaemons/com.vagrant.vagrant-vmware-utility.plist
```

### Hostname Resolution Not Working

```
# Reprovision to update /etc/hosts
vagrant reload --provision
```

### Check VMware Utility Status

```
sudo launchctl list | grep vagrant-vmware-utility
curl -k https://127.0.0.1:9922/api/status
```

## Clean Up

To completely remove the environment:

```
# Destroy all VMs
vagrant destroy -f

# Remove Vagrant boxes (optional)
vagrant box remove bento/ubuntu-24.04

# Remove .vagrant directory
rm -rf .vagrant
```

## System Requirements

- **Disk Space**: ~15GB (5GB per VM)
- **RAM**: 6GB minimum (2GB per VM)
- **CPU**: Multi-core processor recommended

## License

This project is provided as-is for educational and testing purposes.

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.

## References

- [JBoss EAP Documentation](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/)
- [Vagrant Documentation](https://developer.hashicorp.com/vagrant/docs)
- [VMware Fusion](https://www.vmware.com/products/fusion.html)

## Author

Created for JBoss EAP clustering and HA testing on Apple Silicon Macs.

