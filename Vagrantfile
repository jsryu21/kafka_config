# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
BOX_IMAGE = "bento/ubuntu-16.04"
ZK_NODE_COUNT = 3
KAFKA_NODE_COUNT = 3

$master_script = <<-SCRIPT
sudo su

cp -p /etc/apt/sources.list /etc/apt/sources.list.bk
sed -i 's/archive.ubuntu.com/ftp.daumkakao.com/g' /etc/apt/sources.list
apt-get update -y
apt-get install openjdk-8-jdk -y
echo JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 >> /etc/environment
source /etc/environment

echo 192.168.0.10 master >> /etc/hosts
for (( c = 1; c <= 9; c++ ))
do
  echo 192.168.0.2$c peter-zk00$c >> /etc/hosts
done
for (( c = 1; c <= 9; c++ ))
do
  echo 192.168.0.3$c peter-kafka00$c >> /etc/hosts
done

apt-get -y install python3-pip
pip3 install kafka-python
pip3 install jupyter

touch /etc/systemd/system/jupyter-server.service
echo '[Unit]
Description=Jupyter Notebook
Requires=local-fs.target
After=local-fs.target

[Service]
ExecStart=/usr/local/bin/jupyter notebook --allow-root --ip=0.0.0.0 --port=8888
User=%i

[Install]
WantedBy=multi-user.target' > /etc/systemd/system/jupyter-server.service
systemctl daemon-reload
systemctl enable jupyter-server.service
systemctl start jupyter-server.service
SCRIPT

$zk_script = <<-SCRIPT
sudo su

cp -p /etc/apt/sources.list /etc/apt/sources.list.bk
sed -i 's/archive.ubuntu.com/ftp.daumkakao.com/g' /etc/apt/sources.list
apt-get update -y
apt-get install openjdk-8-jdk -y
echo JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 >> /etc/environment
source /etc/environment

echo 192.168.0.10 master >> /etc/hosts
for (( c = 1; c <= 9; c++ ))
do
  echo 192.168.0.2$c peter-zk00$c >> /etc/hosts
done
for (( c = 1; c <= 9; c++ ))
do
  echo 192.168.0.3$c peter-kafka00$c >> /etc/hosts
done

cd /usr/local/
wget -q http://apache.mirror.cdnetworks.com/zookeeper/zookeeper-3.4.10/zookeeper-3.4.10.tar.gz
tar zxf zookeeper-3.4.10.tar.gz
ln -s zookeeper-3.4.10 zookeeper

mkdir -p /data
echo $1 > /data/myid

touch /usr/local/zookeeper/conf/zoo.cfg
echo 'tickTime=2000
initLimit=10
syncLimit=5
dataDir=/data
clientPort=2181' > /usr/local/zookeeper/conf/zoo.cfg
for (( c = 1; c <= $2; c++ ))
do
  echo server.$c=192.168.0.2$c:2888:3888 >> /usr/local/zookeeper/conf/zoo.cfg
done

touch /etc/systemd/system/zookeeper-server.service
echo '[Unit]
Description=zookeeper-server
After=network.target

[Service]
Type=forking
User=root
Group=root
SyslogIdentifier=zookeeper-server
WorkingDirectory=/usr/local/zookeeper
Restart=always
RestartSec=0s
ExecStart=/usr/local/zookeeper/bin/zkServer.sh start
ExecStop=/usr/local/zookeeper/bin/zkServer.sh stop

[Install]
WantedBy=multi-user.target' > /etc/systemd/system/zookeeper-server.service
systemctl daemon-reload
systemctl enable zookeeper-server.service
systemctl start zookeeper-server.service
SCRIPT

$kafka_script = <<-SCRIPT
sudo su

cp -p /etc/apt/sources.list /etc/apt/sources.list.bk
sed -i 's/archive.ubuntu.com/ftp.daumkakao.com/g' /etc/apt/sources.list
apt-get update -y
apt-get install openjdk-8-jdk -y
echo JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 >> /etc/environment
source /etc/environment

echo 192.168.0.10 master >> /etc/hosts
for (( c = 1; c <= 9; c++ ))
do
  echo 192.168.0.2$c peter-zk00$c >> /etc/hosts
done
for (( c = 1; c <= 9; c++ ))
do
  echo 192.168.0.3$c peter-kafka00$c >> /etc/hosts
done

cd /usr/local/
wget -q http://apache.mirror.cdnetworks.com/kafka/1.0.2/kafka_2.11-1.0.2.tgz
tar zxf kafka_2.11-1.0.2.tgz
ln -s kafka_2.11-1.0.2 kafka

mkdir -p /data1
mkdir -p /data2

cp -p /usr/local/kafka/bin/kafka-server-start.sh /usr/local/kafka/bin/kafka-server-start.sh.bk
sed -i -e '17 i\
export JMX_PORT=9999' /usr/local/kafka/bin/kafka-server-start.sh

touch /usr/local/kafka/config/jmx
echo JMX_PORT=9999 > /usr/local/kafka/config/jmx

cp -p /usr/local/kafka/config/server.properties /usr/local/kafka/config/server.properties.bk
sed -i "s/broker.id=0/broker.id=$1/" /usr/local/kafka/config/server.properties
sed -i 's|log.dirs=/tmp/kafka-logs|log.dirs=/data1,/data2|' /usr/local/kafka/config/server.properties
sed -i 's|zookeeper.connect=localhost:2181|zookeeper.connect=peter-zk001:2181,peter-zk002:2181,peter-zk003:2181/peter-kafka|' /usr/local/kafka/config/server.properties

touch /etc/systemd/system/kafka-server.service
echo '[Unit]
Description=kafka-server
After=network.target

[Service]
Type=simple
User=root
Group=root
SyslogIdentifier=kafka-server
WorkingDirectory=/usr/local/kafka
Restart=no
RestartSec=0s
ExecStart=/usr/local/kafka/bin/kafka-server-start.sh /usr/local/kafka/config/server.properties
ExecStop=/usr/local/kafka/bin/kafka-server-stop.sh
EnvironmentFile=/usr/local/kafka/config/jmx' > /etc/systemd/system/kafka-server.service
systemctl daemon-reload
systemctl enable kafka-server.service
systemctl start kafka-server.service
SCRIPT

$kafka_manager_script = <<-SCRIPT
sudo su

apt-get install -y unzip

cd /opt/
wget https://github.com/yahoo/kafka-manager/archive/1.3.3.18.zip
unzip 1.3.3.18.zip
cd /opt/kafka-manager-1.3.3.18
./sbt clean dist

cp /opt/kafka-manager-1.3.3.18/target/universal/kafka-manager-1.3.3.18.zip /usr/local/
cd /usr/local/
unzip kafka-manager-1.3.3.18.zip

cp -p /usr/local/kafka-manager-1.3.3.18/conf/application.conf /usr/local/kafka-manager-1.3.3.18/conf/application.conf.bk
sed -i 's/kafka-manager.zkhosts="kafka-manager-zookeeper:2181"/kafka-manager.zkhosts="peter-zk001:2181,peter-zk002:2181,peter-zk003:2181"/' /usr/local/kafka-manager-1.3.3.18/conf/application.conf

#/usr/local/kafka-manager-1.3.3.18/bin/kafka-manager -Dconfig.file=/usr/local/kafka-manager-1.3.3.18/conf/application.conf -Dapplication.home=/usr/local/kafka-manager-1.3.3.18 -Dhttp.port=9000 -Djava.net.preferIPv4Stack=true
SCRIPT

$filebeat_script = <<-SCRIPT
sudo su

wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list
apt-get update
apt-get install -y filebeat

cp -p /etc/filebeat/filebeat.yml /etc/filebeat/filebeat.yml.bk
echo '#======= Filebeat prospectors
kafka.home: /usr/local/kafka
filebeat.prospectors:
- input_type: log
  paths:
    - ${kafka.home}/logs/server.log*
  multiline.pattern: '"'"'^\\['"'"'
  multiline.negate: true
  multiline.match: after
  fields.pipeline: kafka-logs
#======= Filebeat modules
filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: false

#====== Elasticsearch template setting

setup.template.settings:
  index.number_of_shards: 3

#========== KAFKA output
output.kafka:
  hosts: ["peter-kafka001:9092", "peter-kafka002:9092", "peter-kafka003:9092"]

  topic: '"'"'peter-log'"'"'
  partition.round_robin:
    reachable_only: false

  required_acks: 1
  compression: gzip
  max_message_bytes: 1000000' > /etc/filebeat/filebeat.yml

systemctl start filebeat.service
SCRIPT

$nifi_script = <<-SCRIPT
sudo su

cd /usr/local/
wget -q http://apache.mirror.cdnetworks.com/nifi/1.8.0/nifi-1.8.0-bin.tar.gz
tar zxf nifi-1.8.0-bin.tar.gz
ln -s nifi-1.8.0 nifi

cp -p /usr/local/nifi/conf/nifi.properties /usr/local/nifi/conf/nifi.properties.bk
sed -i 's/nifi.web.http.host=/nifi.web.http.host=0.0.0.0/' /usr/local/nifi/conf/nifi.properties
sed -i 's/nifi.zookeeper.connect.string=/nifi.zookeeper.connect.string=peter-zk001:2181,peter-zk002:2181,peter-zk003:2181/' /usr/local/nifi/conf/nifi.properties

/usr/local/nifi/bin/nifi.sh install nifi

systemctl daemon-reload
systemctl start nifi.service
SCRIPT

$elasticsearch_script = <<-SCRIPT
sudo su

wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://packages.elastic.co/elasticsearch/2.x/debian stable main" | sudo tee -a /etc/apt/sources.list.d/elasticsearch-2.x.list
apt-get update
apt-get install -y elasticsearch

cp -p /etc/elasticsearch/elasticsearch.yml /etc/elasticsearch/elasticsearch.yml.bk
echo 'path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
cluster.name: peter-es
node.name: peter-kafka001
network.bind_host: 0.0.0.0
http.port: 9200
transport.tcp.port: 9300' > /etc/elasticsearch/elasticsearch.yml

systemctl start elasticsearch.service
SCRIPT

$kibana_script = <<-SCRIPT
sudo su

wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://packages.elastic.co/kibana/4.6/debian stable main" | sudo tee -a /etc/apt/sources.list.d/kibana.list
apt-get update
apt-get install -y kibana

cp -p /etc/kibana/kibana.yml /etc/kibana/kibana.yml.bk
echo 'server.host: "0.0.0.0"
elasticsearch.url: "http://0.0.0.0:9200"' > /etc/kibana/kibana.yml

systemctl start kibana.service
SCRIPT

Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  #config.vm.box = "base"
  config.vm.box = BOX_IMAGE

  config.vm.define "master" do |subconfig|
    subconfig.vm.network "forwarded_port", guest: 8888, host: 8888, host_ip: "127.0.0.1"
    subconfig.vm.network "private_network", ip: "192.168.0.10"
    subconfig.vm.hostname = "master"
    subconfig.vm.provider :virtualbox do |vb|
      vb.gui = false
      vb.customize ["modifyvm", :id, "--memory", "1024"]
      vb.customize ["modifyvm", :id, "--cpus", "1"]
    end
    subconfig.vm.provision "shell" do |s|
      s.inline = $master_script
    end
  end

  (1..ZK_NODE_COUNT).each do |i|
    config.vm.define "peter-zk00#{i}" do |subconfig|
      subconfig.vm.network "private_network", ip: "192.168.0.2#{i}"
      subconfig.vm.hostname = "peter-zk00#{i}"
      subconfig.vm.provider :virtualbox do |vb|
        vb.gui = false
        vb.customize ["modifyvm", :id, "--memory", "1024"]
        vb.customize ["modifyvm", :id, "--cpus", "1"]
      end
      subconfig.vm.provision "shell" do |s|
        s.inline = $zk_script
        s.args   = ["#{i}", ZK_NODE_COUNT]
      end
    end
  end

  config.vm.define "peter-kafka002" do |subconfig|
    subconfig.vm.network "private_network", ip: "192.168.0.32"
    subconfig.vm.hostname = "peter-kafka002"
    subconfig.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "1024"]
      vb.customize ["modifyvm", :id, "--cpus", "1"]
    end
    subconfig.vm.provision "shell" do |s|
      s.inline = $kafka_script
      s.args   = ["2"]
    end
  end

  config.vm.define "peter-kafka003" do |subconfig|
    subconfig.vm.network "private_network", ip: "192.168.0.33"
    subconfig.vm.hostname = "peter-kafka003"
    subconfig.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "1024"]
      vb.customize ["modifyvm", :id, "--cpus", "1"]
    end
    subconfig.vm.provision "shell" do |s|
      s.inline = $kafka_script
      s.args   = ["3"]
    end
  end

  config.vm.define "peter-kafka001" do |subconfig|
    # kafka-manager
    subconfig.vm.network "forwarded_port", guest: 9000, host: 9000, host_ip: "127.0.0.1"
    # nifi
    subconfig.vm.network "forwarded_port", guest: 8080, host: 8080, host_ip: "127.0.0.1"
    # elasticsearch
    subconfig.vm.network "forwarded_port", guest: 9200, host: 9200, host_ip: "127.0.0.1"
    # kibana
    subconfig.vm.network "forwarded_port", guest: 5601, host: 5601, host_ip: "127.0.0.1"
    subconfig.vm.network "private_network", ip: "192.168.0.31"
    subconfig.vm.hostname = "peter-kafka001"
    subconfig.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "4096"]
      vb.customize ["modifyvm", :id, "--cpus", "2"]
    end
    subconfig.vm.provision "kafka",
      type: "shell",
      preserve_order: true,
      inline: $kafka_script,
      args: ["1"]
    subconfig.vm.provision "filebeat",
      type: "shell",
      preserve_order: true,
      inline: $filebeat_script
    subconfig.vm.provision "nifi",
      type: "shell",
      preserve_order: true,
      inline: $nifi_script
    subconfig.vm.provision "elasticsearch",
      type: "shell",
      preserve_order: true,
      inline: $elasticsearch_script
    subconfig.vm.provision "kibana",
      type: "shell",
      preserve_order: true,
      inline: $kibana_script
    #subconfig.vm.provision "kafka_manager",
    #  type: "shell",
    #  preserve_order: true,
    #  inline: $kafka_manager_script
  end

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "2048"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
end
