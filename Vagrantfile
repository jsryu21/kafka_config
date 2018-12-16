# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
BOX_IMAGE = "bento/ubuntu-16.04"
ZK_NODE_IDX = 1
ZK_NODE_COUNT = 3
KAFKA_NODE_IDX = 1
KAFKA_NODE_COUNT = 3

$master_script = <<-SCRIPT
sudo su
cd
apt-get update -y
#apt-get upgrade -y
apt-get install openjdk-8-jdk -y

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
jupyter notebook --allow-root --ip=0.0.0.0 --port=8888
SCRIPT

$zk_script = <<-SCRIPT
sudo su
cd
apt-get update -y
#apt-get upgrade -y
apt-get install openjdk-8-jdk -y

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
wget http://apache.mirror.cdnetworks.com/zookeeper/zookeeper-3.4.10/zookeeper-3.4.10.tar.gz
tar zxf zookeeper-3.4.10.tar.gz
ln -s zookeeper-3.4.10 zookeeper

cd
mkdir -p /data
echo $1 > /data/myid

touch /usr/local/zookeeper/conf/zoo.cfg
echo tickTime=2000 >> /usr/local/zookeeper/conf/zoo.cfg
echo initLimit=10 >> /usr/local/zookeeper/conf/zoo.cfg
echo syncLimit=5 >> /usr/local/zookeeper/conf/zoo.cfg
echo dataDir=/data >> /usr/local/zookeeper/conf/zoo.cfg
echo clientPort=2181 >> /usr/local/zookeeper/conf/zoo.cfg
for (( c = 1; c <= $2; c++ ))
do
  echo server.$c=192.168.0.2$c:2888:3888 >> /usr/local/zookeeper/conf/zoo.cfg
done

cd /etc/systemd/system/
wget https://raw.githubusercontent.com/jsryu21/kafka_config/master/zookeeper-server.service
systemctl daemon-reload
systemctl enable zookeeper-server.service
SCRIPT

$kafka_script = <<-SCRIPT
sudo su
cd
apt-get update -y
#apt-get upgrade -y
apt-get install openjdk-8-jdk -y

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
wget http://apache.mirror.cdnetworks.com/kafka/1.0.2/kafka_2.11-1.0.2.tgz
tar zxf kafka_2.11-1.0.2.tgz
ln -s kafka_2.11-1.0.2 kafka

cd
mkdir -p /data1
mkdir -p /data2

cd /usr/local/kafka/bin/
mv kafka-server-start.sh kafka-server-start.sh.bk
wget https://raw.githubusercontent.com/jsryu21/kafka_config/master/kafka-server-start.sh
chmod +x kafka-server-start.sh

cd /usr/local/kafka/config/
wget https://raw.githubusercontent.com/jsryu21/kafka_config/master/jmx

cd /usr/local/kafka/config/
wget https://raw.githubusercontent.com/jsryu21/kafka_config/master/server$1.properties

cd /etc/systemd/system/
wget https://raw.githubusercontent.com/jsryu21/kafka_config/master/kafka-server$1.service
systemctl daemon-reload
systemctl enable kafka-server$1.service
SCRIPT

$zk_start_script = <<-SCRIPT
sudo su
cd
systemctl start zookeeper-server.service
SCRIPT

$kafka_start_script = <<-SCRIPT
sudo su
cd
systemctl start kafka-server$1.service
SCRIPT

$kafka001_start_script = <<-SCRIPT
sudo su
apt-get install -y unzip
cd
wget https://github.com/yahoo/kafka-manager/archive/1.3.3.18.zip
unzip 1.3.3.18.zip
cd /root/kafka-manager-1.3.3.18/
./sbt clean dist
cp /root/kafka-manager-1.3.3.18/target/universal/kafka-manager-1.3.3.18.zip /usr/local/
cd /usr/local/
unzip kafka-manager-1.3.3.18.zip
cd /usr/local/kafka-manager-1.3.3.18/conf/
mv application.conf application.conf.bk
wget https://raw.githubusercontent.com/jsryu21/kafka_config/master/application.conf
cd /usr/local/kafka-manager-1.3.3.18/bin/
#./kafka-manager -Dconfig.file=/usr/local/kafka-manager-1.3.3.18/conf/application.conf -Dapplication.home=/usr/local/kafka-manager-1.3.3.18 -Dhttp.port=9000 -Djava.net.preferIPv4Stack=true
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
    subconfig.vm.network "private_network", ip: "192.168.0.10"
	subconfig.vm.network "forwarded_port", guest: 8888, host: 8888, host_ip: "127.0.0.1"
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

  (ZK_NODE_IDX..ZK_NODE_COUNT).each do |i|
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

  (KAFKA_NODE_IDX..KAFKA_NODE_COUNT).each do |i|
    config.vm.define "peter-kafka00#{i}" do |subconfig|
      subconfig.vm.network "private_network", ip: "192.168.0.3#{i}"
      subconfig.vm.hostname = "peter-kafka00#{i}"
      subconfig.vm.provider :virtualbox do |vb|
        vb.gui = false
        vb.customize ["modifyvm", :id, "--memory", "1024"]
        vb.customize ["modifyvm", :id, "--cpus", "1"]
      end
      subconfig.vm.provision "shell" do |s|
        s.inline = $kafka_script
        s.args   = ["#{i}"]
      end
    end
  end

  (ZK_NODE_IDX..ZK_NODE_COUNT).each do |i|
    config.vm.define "peter-zk00#{i}" do |subconfig|
      subconfig.vm.provision "shell" do |s|
        s.inline = $zk_start_script
      end
    end
  end

  (KAFKA_NODE_IDX..ZK_NODE_COUNT).each do |i|
    config.vm.define "peter-kafka00#{i}" do |subconfig|
      subconfig.vm.provision "shell" do |s|
        s.inline = $kafka_start_script
        s.args   = ["#{i}"]
      end
    end
  end

  config.vm.define "peter-kafka001" do |subconfig|
    subconfig.vm.network "forwarded_port", guest: 9000, host: 9000, host_ip: "127.0.0.1"
    subconfig.vm.provision "shell" do |s|
      s.inline = $kafka001_start_script
    end
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
