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
    cat /etc/hosts
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
    
    # Provision Java and hosts
    node1.vm.provision "shell", inline: $install_java
    node1.vm.provision "shell", inline: $configure_hosts
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
    
    # Provision Java and hosts
    node2.vm.provision "shell", inline: $install_java
    node2.vm.provision "shell", inline: $configure_hosts
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
    
    # Provision Java and hosts
    node3.vm.provision "shell", inline: $install_java
    node3.vm.provision "shell", inline: $configure_hosts
  end
end

