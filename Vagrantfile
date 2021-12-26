# encoding: utf-8
# -*- mode: ruby -*-
# vi: set ft=ruby :

NODE_SCRIPT = <<EOF.freeze
echo "Preparing node..."

# Generate a new root key
openssl genrsa -out /vagrant/rootcerts/rootCA.key 2048

# Generate a root certificate (enter anything at the prompts)
openssl req -x509 -new -nodes -key /vagrant/rootcerts/rootCA.key -days 10000 -out /vagrant/rootcerts/rootCA.crt -subj "/C=FR/ST=France/L=Paris/O=Slim Company Name/OU=Org/CN=www.slim.com"

# Generate a key for your docker-registry server
openssl genrsa -out /vagrant/nginx/certs/docker-registry.key 2048

# Make a certificate signing request 
# for "Common Name" make sure to type in the domain your server, in this case its docker-registry.local
# do not enter a challenge password
openssl req -new -key /vagrant/nginx/certs/docker-registry.key -out /vagrant/nginx/certs/docker-registry.local.csr -config /vagrant/nginx/certs/sslconf.ini

# Sign the certficate
openssl x509 -req -extfile <(printf "subjectAltName=DNS: docker-registry.local,DNS:docker-registry,IP:10.1.1.30") -in /vagrant/nginx/certs/docker-registry.local.csr -CA /vagrant/rootcerts/rootCA.crt -CAkey /vagrant/rootcerts/rootCA.key -CAcreateserial -out /vagrant/nginx/certs/docker-registry.crt -days 365

# Install docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Install docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Start & enable docker
systemctl start docker
systemctl enable docker

# Verify docker installation
docker run --rm hello-world

# Copy docker-registry.service into systemd
cp /vagrant/docker-registry/service/docker-registry.service /etc/systemd/system/

# Enable and start docker-registry service
systemctl enable docker-registry.service
service docker-registry start


#
# TO TEST CLIENT IN LOCAL : 

sudo echo "10.1.1.30 docker-registry.local" >> /etc/hosts
if [ -f /etc/redhat-release ]; then
	#Centos clients
	sudo cp /vagrant/rootcerts/rootCA.crt /etc/pki/ca-trust/source/anchors/
	sudo update-ca-trust extract
fi

if [ -f /etc/lsb-release ]; then
	#Ubuntu clients
	sudo cp /vagrant/rootcerts/rootCA.crt /usr/local/share/ca-certificates/
	sudo update-ca-certificates
fi


sudo systemctl restart docker
sleep 10
sudo docker login -u admin -p admin https://docker-registry.local

EOF

def set_hostname(server)
  server.vm.provision 'shell', inline: "hostname #{server.vm.hostname}"
end

Vagrant.configure(2) do |config|
  config.vm.define 'docker-registry' do |d|
    d.vm.box = 'bento/ubuntu-20.04'
	#d.vm.box = 'centos/7'
    d.vm.hostname = 'docker-registry'
    d.vm.network 'private_network', ip: '10.1.1.30'
    d.vm.provision :shell, inline: NODE_SCRIPT.dup
    set_hostname(d)

    d.vm.provider 'virtualbox' do |v|
      v.memory = 1024
      v.cpus = 1
    end
  end
end
