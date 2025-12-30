# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-24.04"
  config.vm.box_version = "202510.26.0"

  config.vm.provider "vmware_desktop" do |vmware|
    vmware.gui = false
    vmware.memory = "2048"
    vmware.cpus = 2
    vmware.linked_clone = false
  end

  # Install Java
  $install_java = <<-SCRIPT
    echo "Installing OpenJDK 17..."
    apt-get update
    apt-get install -y openjdk-17-jdk
    
    # Set JAVA_HOME
    echo 'export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-arm64' >> /etc/profile.d/java.sh
    echo 'export PATH=$JAVA_HOME/bin:$PATH' >> /etc/profile.d/java.sh
    
    # Verify installation
    java -version
    echo "Java installation complete!"
  SCRIPT

  # Configure /etc/hosts for all nodes
  $configure_hosts = <<-SCRIPT
    echo "Configuring /etc/hosts for cluster communication..."
    
    # Remove any existing entries for our hostnames
    sed -i '/jboss-node/d' /etc/hosts
    
    # Add all cluster nodes
    cat >> /etc/hosts <<EOF

# JBoss EAP Cluster Nodes
192.168.56.11 jboss-node1
192.168.56.12 jboss-node2
192.168.56.13 jboss-node3
EOF
    
    echo "Hostname resolution configured!"
  SCRIPT

  # Setup SSH keys - this runs on all nodes
  $setup_ssh = <<-SCRIPT
    echo "Setting up SSH for passwordless access..."
    
    # Enable root login
    sed -i 's/^#*PermitRootLogin.*/PermitRootLogin prohibit-password/' /etc/ssh/sshd_config
    sed -i 's/^PasswordAuthentication.*/PasswordAuthentication yes/' /etc/ssh/sshd_config
    systemctl restart ssh
    
    # Create SSH directories
    mkdir -p /root/.ssh
    chmod 700 /root/.ssh
    mkdir -p /home/vagrant/.ssh
    chmod 700 /home/vagrant/.ssh
    chown vagrant:vagrant /home/vagrant/.ssh
    
    # Create a shared SSH key pair for the cluster
    # This is the private key (same for all nodes)
    cat > /root/.ssh/id_rsa <<'SSHKEY'
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEAzXrCLfvv5qXKJlL5RHCqJKUGCmB0K8M8R5PvPIxvF8HQd8LvNfDK
8KN3h7ZnYvPJZxvYL5lK3PvH2YmFH9VzZK7YQG8xQY5Bv7Z8L3PvH2YmFH9VzZK7YQG8
xQY5Bv7Z8L3PvH2YmFH9VzZK7YQG8xQY5Bv7Z8L3PvH2YmFH9VzZK7YQG8xQY5Bv7Z8L
3PvH2YmFH9VzZK7YQG8xQY5Bv7Z8L3PvH2YmFH9VzZK7YQG8xQY5Bv7Z8L3PvH2YmFH9
VzZK7YQG8xQY5Bv7Z8L3PvH2YmFH9VzZK7YQG8xQY5Bv7Z8L3PvH2YmFH9VzZK7YQG8x
QY5Bv7Z8L3PvH2YmFH9VzZK7YQG8xQY5Bv7Z8L3PvH2YmFH9VzZK7YQG8xQY5Bv7Z8L3
PvH2YmFH9VzZK7YQG8xQY5Bv7Z8L3PvH2YmFH9VzZK7YAAAA
-----END OPENSSH PRIVATE KEY-----
SSHKEY

    # This is a simpler approach: generate a new key on first node
    if [ ! -f /vagrant/.ssh/cluster_rsa ]; then
      mkdir -p /vagrant/.ssh
      ssh-keygen -t rsa -b 2048 -N "" -f /vagrant/.ssh/cluster_rsa -C "cluster-key"
    fi
    
    # Copy the shared keys
    cp /vagrant/.ssh/cluster_rsa /root/.ssh/id_rsa
    cp /vagrant/.ssh/cluster_rsa.pub /root/.ssh/id_rsa.pub
    cp /vagrant/.ssh/cluster_rsa /home/vagrant/.ssh/id_rsa
    cp /vagrant/.ssh/cluster_rsa.pub /home/vagrant/.ssh/id_rsa.pub
    
    # Set correct permissions
    chmod 600 /root/.ssh/id_rsa
    chmod 644 /root/.ssh/id_rsa.pub
    chmod 600 /home/vagrant/.ssh/id_rsa
    chmod 644 /home/vagrant/.ssh/id_rsa.pub
    chown vagrant:vagrant /home/vagrant/.ssh/id_rsa*
    
    # Add the public key to authorized_keys
    cat /vagrant/.ssh/cluster_rsa.pub >> /root/.ssh/authorized_keys
    cat /vagrant/.ssh/cluster_rsa.pub >> /home/vagrant/.ssh/authorized_keys
    chmod 600 /root/.ssh/authorized_keys
    chmod 600 /home/vagrant/.ssh/authorized_keys
    chown vagrant:vagrant /home/vagrant/.ssh/authorized_keys
    
    # Configure SSH to not check host keys within cluster
    cat > /root/.ssh/config <<EOF
Host 192.168.56.* jboss-node*
    StrictHostKeyChecking no
    UserKnownHostsFile=/dev/null
    LogLevel ERROR
EOF
    chmod 600 /root/.ssh/config
    
    cat > /home/vagrant/.ssh/config <<EOF
Host 192.168.56.* jboss-node*
    StrictHostKeyChecking no
    UserKnownHostsFile=/dev/null
    LogLevel ERROR
EOF
    chmod 600 /home/vagrant/.ssh/config
    chown vagrant:vagrant /home/vagrant/.ssh/config
    
    echo "SSH setup complete!"
  SCRIPT

  # Define node1
  config.vm.define "node1" do |node1|
    node1.vm.hostname = "jboss-node1"
    node1.vm.network "private_network", ip: "192.168.56.11"

    node1.vm.network "forwarded_port", guest: 22, host: 2201
    node1.vm.network "forwarded_port", guest: 80, host: 8001
    node1.vm.network "forwarded_port", guest: 443, host: 8431
    node1.vm.network "forwarded_port", guest: 8080, host: 8081
    node1.vm.network "forwarded_port", guest: 8443, host: 8444
    node1.vm.network "forwarded_port", guest: 8990, host: 8991
    node1.vm.network "forwarded_port", guest: 9990, host: 9991

    node1.vm.provider "vmware_desktop" do |vmware|
      vmware.vmx["displayname"] = "jboss-node1"
    end
    
    node1.vm.provision "shell", inline: $install_java
    node1.vm.provision "shell", inline: $configure_hosts
    node1.vm.provision "shell", inline: $setup_ssh
  end

  # Define node2
  config.vm.define "node2" do |node2|
    node2.vm.hostname = "jboss-node2"
    node2.vm.network "private_network", ip: "192.168.56.12"

    node2.vm.network "forwarded_port", guest: 22, host: 2202
    node2.vm.network "forwarded_port", guest: 80, host: 8002
    node2.vm.network "forwarded_port", guest: 443, host: 8432
    node2.vm.network "forwarded_port", guest: 8080, host: 8082
    node2.vm.network "forwarded_port", guest: 8443, host: 8445
    node2.vm.network "forwarded_port", guest: 8990, host: 8992
    node2.vm.network "forwarded_port", guest: 9990, host: 9992

    node2.vm.provider "vmware_desktop" do |vmware|
      vmware.vmx["displayname"] = "jboss-node2"
    end
    
    node2.vm.provision "shell", inline: $install_java
    node2.vm.provision "shell", inline: $configure_hosts
    node2.vm.provision "shell", inline: $setup_ssh
  end

  # Define node3
  config.vm.define "node3" do |node3|
    node3.vm.hostname = "jboss-node3"
    node3.vm.network "private_network", ip: "192.168.56.13"

    node3.vm.network "forwarded_port", guest: 22, host: 2203
    node3.vm.network "forwarded_port", guest: 80, host: 8003
    node3.vm.network "forwarded_port", guest: 443, host: 8433
    node3.vm.network "forwarded_port", guest: 8080, host: 8083
    node3.vm.network "forwarded_port", guest: 8443, host: 8446
    node3.vm.network "forwarded_port", guest: 8990, host: 8993
    node3.vm.network "forwarded_port", guest: 9990, host: 9993

    node3.vm.provider "vmware_desktop" do |vmware|
      vmware.vmx["displayname"] = "jboss-node3"
    end
    
    node3.vm.provision "shell", inline: $install_java
    node3.vm.provision "shell", inline: $configure_hosts
    node3.vm.provision "shell", inline: $setup_ssh
  end
end


