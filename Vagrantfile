# -*- mode: ruby -*-
# vi: set ft=ruby :

$vault = <<VAULT
echo "Installing vault ..."
sudo apt-get update
sudo apt-get install curl unzip -y
cd /tmp/
curl -s https://releases.hashicorp.com/vault/${VAULT_DEMO_VERSION}/vault_${VAULT_DEMO_VERSION}_linux_amd64.zip -o vault.zip
unzip vault.zip
sudo chmod +x vault
sudo mv vault /usr/bin/vault
sudo mkdir /etc/vault
sudo chmod a+w /etc/vault
VAULT

$consul = <<CONSUL
echo "Installing Consul ..."
sudo apt-get update
sudo apt-get install -y unzip curl jq dnsutils
echo "Fetching Consul version ${CONSUL_DEMO_VERSION} ..."
cd /tmp/
curl -s https://releases.hashicorp.com/consul/${CONSUL_DEMO_VERSION}/consul_${CONSUL_DEMO_VERSION}_linux_amd64.zip -o consul.zip
echo "Installing Consul version ${CONSUL_DEMO_VERSION} ..."
unzip consul.zip
sudo chmod +x consul
sudo mv consul /usr/bin/consul
sudo mkdir /etc/consul.d
sudo chmod a+w /etc/consul.d
CONSUL

$n1 = <<N1
sudo cp /vagrant/consul-server.json /etc/consul.d
nohup consul agent -config-file=/etc/consul.d/consul-server.json \
 -bind "172.20.20.13" -retry-join "172.20.20.14" -retry-join "172.20.20.15" > \
  /tmp/consul-out.txt 2> /tmp/consul-err.txt &
sudo cp /vagrant/vault-n1.hcl /etc/vault
sleep 10
sudo nohup vault server -config=/etc/vault/vault-n1.hcl -log-level=debug &
sudo cat <<EOF > /etc/profile.d/vault.sh
export VAULT_ADDR="http://127.0.0.1:8200"
export VAULT_TOKEN="$(cat /vagrant/root_token)"
export VAULT_SKIP_VERIFY=true
EOF

N1

$n2 = <<N2
sudo cp /vagrant/consul-server.json /etc/consul.d
nohup consul agent -config-file=/etc/consul.d/consul-server.json \
 -bind "172.20.20.14" -retry-join "172.20.20.13" -retry-join "172.20.20.15" > \
  /tmp/consul-out.txt 2> /tmp/consul-err.txt &
sudo cp /vagrant/vault-n2.hcl /etc/vault
sleep 10
sudo nohup vault server -config=/etc/vault/vault-n2.hcl -log-level=debug &
sudo cat <<EOF > /etc/profile.d/vault.sh
export VAULT_ADDR="http://127.0.0.1:8200"
export VAULT_SKIP_VERIFY=true
EOF

N2

$n3 = <<N3
sudo cp /vagrant/consul-server.json /etc/consul.d
nohup consul agent -config-file=/etc/consul.d/consul-server.json \
 -bind "172.20.20.15" -retry-join "172.20.20.14" -retry-join "172.20.20.13" > \
  /tmp/consul-out.txt 2> /tmp/consul-err.txt &
sudo cat <<EOF > /etc/profile.d/vault.sh
export VAULT_ADDR="http://172.20.20.14:8200"
export VAULT_SKIP_VERIFY=true
EOF

N3

$hosts_file = <<HOSTS_FILE
sudo cat << EOF >> /etc/hosts
172.20.20.13  n1 n1.example.com
172.20.20.14  n2 n2.example.com
172.20.20.15  n3 n3.example.com
EOF
HOSTS_FILE

# Specify a Consul version
CONSUL_DEMO_VERSION = ENV['CONSUL_DEMO_VERSION'] || "1.2.2"

# Specify a Vault version
VAULT_DEMO_VERSION = ENV['VAULT_DEMO_VERSION'] || "0.10.4"

# Specify a custom Vagrant box for the demo
DEMO_BOX_NAME = ENV['DEMO_BOX_NAME'] || "debian/stretch64"

# Vagrantfile API/syntax version.
# NB: Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = DEMO_BOX_NAME

  config.vm.provider "virtualbox" do |vb|
     vb.memory = "256"
  end

  config.vm.provision "shell",
                          inline: $vault,
                          env: {'VAULT_DEMO_VERSION' => VAULT_DEMO_VERSION}
  config.vm.provision "shell",
                          inline: $consul,
                          env: {'CONSUL_DEMO_VERSION' => CONSUL_DEMO_VERSION}

  config.vm.define "n1" do |n1|
      n1.vm.hostname = "n1"
      n1.vm.network "private_network", ip: "172.20.20.13"
      n1.vm.provision "shell", inline: $hosts_file
      n1.vm.provision "shell", inline: $n1

  end

  config.vm.define "n2" do |n2|
      n2.vm.hostname = "n2"
      n2.vm.network "private_network", ip: "172.20.20.14"
      n2.vm.provision "shell", inline: $hosts_file
      n2.vm.provision "shell", inline: $n2
  end

  config.vm.define "n3" do |n3|
      n3.vm.hostname = "n3"
      n3.vm.network "private_network", ip: "172.20.20.15"
      n3.vm.provision "shell", inline: $hosts_file
      n3.vm.provision "shell", inline: $n3
  end
end
