require "yaml"
vagrant_root = File.dirname(File.expand_path(__FILE__))
setting = YAML.load_file("#{vagrant_root}/setting.yaml")

IP_SECTIONS = setting["network"]["control_ip"].match(/^([0-9.]+\.)([^.]+)$/)
# First 3 octets including the trailing dot:
IP_NW = IP_SECTIONS.captures[0]
# Last octet excluding all dots:
IP_START = Integer(IP_SECTIONS.captures[1])
NUM_WORKER_NODES = setting["nodes"]["workers"]["count"]
DNS_SERVERS1 = setting["network"]["dns_servers"][0]
DNS_SERVERS2 = setting["network"]["dns_servers"][1]

Vagrant.configure("2") do |config|
  config.vm.box_check_update = false
  config.vm.provision "shell", env: { "IP_NW" => IP_NW, "IP_START" => IP_START, "NUM_WORKER_NODES" => NUM_WORKER_NODES, "DNS_SERVERS1" => DNS_SERVERS1, "DNS_SERVERS2" => DNS_SERVERS2  }, inline: <<-SHELL
      echo "$IP_NW$((IP_START)) master-node.local master_node" >> /etc/hosts
      temp_file=$(mktemp)
      echo "nameserver $DNS_SERVERS1" > "$temp_file"
      echo "nameserver $DNS_SERVERS2" >> "$temp_file"
      cat /etc/resolv.conf >> "$temp_file"
      mv "$temp_file" /etc/resolv.conf
      for i in `seq 1 ${NUM_WORKER_NODES}`; do
        echo "$IP_NW$((IP_START+i)) worker-node${i}.local worker_node${i}" >> /etc/hosts
      done
      apt-get update -y
  SHELL

  # Master node
  config.vm.define "master_node" do |master_node|
    master_node.vm.hostname = "master-node.local"
    master_node.vm.box = "bento/ubuntu-22.04"
    master_node.vm.network :private_network, ip: "192.168.56.101"

    master_node.vm.provision "shell",
      path: "scripts/common.sh"
  end  # Corrected this line by removing the extra comma

  # Worker node 1
  config.vm.define "worker_node1" do |worker_node1|
    worker_node1.vm.hostname = "worker-node1.local"
    worker_node1.vm.box = "bento/ubuntu-22.04"
    worker_node1.vm.network :private_network, ip: "192.168.56.102"

    worker_node1.vm.provision "shell",
      path: "scripts/common.sh"
  end

  # Worker node 2
  config.vm.define "worker_node2" do |worker_node2|
    worker_node2.vm.hostname = "worker-node2.local"
    worker_node2.vm.box = "bento/ubuntu-22.04"
    worker_node2.vm.network :private_network, ip: "192.168.56.103"
    worker_node2.vm.provision "shell",
      path: "scripts/common.sh"
  end

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus = 2
  end
end