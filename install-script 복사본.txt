These are the commands for AWS CM install
** update yum
sudo yum -y update
sudo yum install -y wget
sudo yum install -y unzip

** add ec2-user to sudoers 

sudo visudo
	add -> ec2-user ALL=(ALL) ALL

** Change the run level to multi-user text mode
sudo systemctl get-default
sudo systemctl set-default multi-user.target

** Disable firewall
sudo systemctl disable firewalld
sudo systemctl status firewalld


** Change VM Swappiness to 1
cat /proc/sys/vm/swappiness
sudo sysctl -w vm.swappiness=1

** Change VM Swappiness permanently
sudo vi /etc/sysctl.conf
	add ->
vm.swappiness=1

** Disable SE Linux
sudo vi /etc/selinux/config 
	change-> SELINUX=disabled


** Disable Transparent Hugepage Support
sudo vi /etc/rc.d/rc.local
	add -> 
echo "never" > /sys/kernel/mm/transparent_hugepage/enabled
echo "never" > /sys/kernel/mm/transparent_hugepage/defrag

sudo chmod +x /etc/rc.d/rc.local
sudo vi /etc/default/grub
   add -> transparent_hugepage=never (on line GRUB_CMDLINE_LINUX )
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
sudo systemctl start tuned
sudo tuned-adm off
sudo tuned-adm list
sudo systemctl stop tuned
sudo systemctl disable tuned

** Check to see that nscd service is running
sudo yum install -y nscd
sudo systemctl enable nscd
sudo systemctl start nscd
sudo systemctl status nscd

** Check to see that ntp service is running
sudo systemctl stop chronyd
sudo systemctl disable chronyd
sudo yum -y install ntp
sudo systemctl enable ntpd.service
sudo systemctl start ntpd.service
sudo ntpdate -u 0.rhel.pool.ntp.org
sudo hwclock --systohc
sudo systemctl status ntpd.service

** check to make sure that File lookup has priority
sudo vi /etc/nsswitch.conf

** Disable IPV6
sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1
sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1

** check hostname resolution
** first put something in /etc/hosts and then

getent hosts cm.bdai.com

** setup a password for ec2-user
sudo passwd ec2-user
sudo vi /etc/ssh/sshd_config
	change ->
PasswordAuthentication yes
sudo systemctl restart sshd.service

** If desired, setup passwordless key entry from CM node
ssh-keygen -t rsa
ssh m1 mkdir -p .ssh
ssh d1 mkdir -p .ssh
ssh d2 mkdir -p .ssh
ssh d3 mkdir -p .ssh
cat .ssh/id_rsa.pub | ssh m1 'cat >> .ssh/authorized_keys'
cat .ssh/id_rsa.pub | ssh d1 'cat >> .ssh/authorized_keys'
cat .ssh/id_rsa.pub | ssh d2 'cat >> .ssh/authorized_keys'
cat .ssh/id_rsa.pub | ssh d3 'cat >> .ssh/authorized_keys'

** Update /etc/host
sudo vi /etc/hosts
	add ->
******* THIS IS PUBLIC IP
13.125.111.228	cm.bdai.com	cm
13.125.61.61	m1.bdai.com	m1
13.209.214.121	d1.bdai.com	d1
13.209.219.244	d2.bdai.com	d2
52.79.64.162	d3.bdai.com	d3

******* THIS IS PRIVATE IP
172.31.12.133	cm.bdai.com	cm
172.31.5.143	m1.bdai.com	m1
172.31.5.226	d1.bdai.com	d1
172.31.8.7	d2.bdai.com	d2
172.31.8.137	d3.bdai.com	d3


** Change the hostname
sudo hostnamectl set-hostname cm.bdai.com
hostname -f

sudo hostnamectl set-hostname m1.bdai.com
hostname -f

sudo hostnamectl set-hostname d1.bdai.com
hostname -f

sudo hostnamectl set-hostname d2.bdai.com
hostname -f

sudo hostnamectl set-hostname d3.bdai.com
hostname -f

** REBOOT THE SERVER AND CHECK

** Install JDK on all machines
// From the Mac
cd ~/Downloads
scp -i ~/KeyPair/SKT.pem ~/Downloads/jdk-8u201-linux-x64.rpm ec2-user@ad3:. &
// From each of the nodes
sudo -i rpm -ivh /home/centos/jdk-8u201-linux-x64.rpm
OR
yum list java*jdk-devel
sudo yum install -y java-1.8.0-openjdk-devel.x86_64
#sudo yum install -y oracle-j2sdk1.8


** Configure repository
sudo yum install -y wget
sudo wget https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/cloudera-manager.repo \
-P /etc/yum.repos.d/

** change the baseurl within cloudera-manager.repo to fit the version you want to install
baseurl=https://archive.cloudera.com/cm5/redhat/6/x86_64/cm/5.7.4/
for example: https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5.14.4/

sudo rpm --import \
https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/RPM-GPG-KEY-cloudera
java -version

** Install Cloudera Manager
sudo yum install -y cloudera-manager-daemons cloudera-manager-server

** Install MariaDB

sudo yum install -y mariadb-server
{ // use this repo in case yum install does not work
sudo vi /etc/yum.repos.d/MariaDB.repo
	add ->

sudo rpm --import https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
}
sudo vi /etc/my.cnf
	add ->
******************************************** Don't add this line
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
transaction-isolation = READ-COMMITTED
# Disabling symbolic-links is recommended to prevent assorted security risks;
# to do so, uncomment this line:
symbolic-links = 0
# Settings user and group are ignored when systemd is used.
# If you need to run mysqld under a different user or group,
# customize your systemd unit file for mariadb according to the
# instructions in http://fedoraproject.org/wiki/Systemd

key_buffer = 16M
key_buffer_size = 32M
max_allowed_packet = 32M
thread_stack = 256K
thread_cache_size = 64
query_cache_limit = 8M
query_cache_size = 64M
query_cache_type = 1

max_connections = 550
#expire_logs_days = 10
#max_binlog_size = 100M

#log_bin should be on a disk with enough free space.
#Replace '/var/lib/mysql/mysql_binary_log' with an appropriate path for your
#system and chown the specified folder to the mysql user.
log_bin=/var/lib/mysql/mysql_binary_log

#In later versions of MariaDB, if you enable the binary log and do not set
#a server_id, MariaDB will not start. The server_id must be unique within
#the replicating group.
server_id=1

binlog_format = mixed

read_buffer_size = 2M
read_rnd_buffer_size = 16M
sort_buffer_size = 8M
join_buffer_size = 8M

# InnoDB settings
innodb_file_per_table = 1
innodb_flush_log_at_trx_commit  = 2
innodb_log_buffer_size = 64M
innodb_buffer_pool_size = 4G
innodb_thread_concurrency = 8
innodb_flush_method = O_DIRECT
innodb_log_file_size = 512M

[mysqld_safe]
log-error=/var/log/mariadb/mariadb.log
pid-file=/var/run/mariadb/mariadb.pid

#
# include all files from the config directory
#
!includedir /etc/my.cnf.d
******************************************** Don't add this line
sudo systemctl enable mariadb
sudo systemctl start mariadb
sudo /usr/bin/mysql_secure_installation

** Install mysql connector
// From the mac
scp -i ~/KeyPair/SEBC_HP.pem ~/Downloads/mysql-connector-java-5.1.47.tar.gz ec2-user@acm:. 
OR
sudo wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.47.tar.gz

// From acm node
cd /home/ec2-user
tar zxvf mysql-connector-java-5.1.47.tar.gz
sudo mkdir -p /usr/share/java/
cd mysql-connector-java-5.1.47
sudo cp mysql-connector-java-5.1.47-bin.jar /usr/share/java/mysql-connector-java.jar

** Create the databases and users in MariaDB
mysql -u root -p

CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON scm.* TO 'scm-user'@'%' IDENTIFIED BY 'somepassword';

CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON amon.* TO 'amon-user'@'%' IDENTIFIED BY 'somepassword';

CREATE DATABASE rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON rman.* TO 'rman-user'@'%' IDENTIFIED BY 'somepassword';

CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON hue.* TO 'hue-user'@'%' IDENTIFIED BY 'somepassword';

CREATE DATABASE metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON metastore.* TO 'metastore-user'@'%' IDENTIFIED BY 'somepassword';

CREATE DATABASE sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON sentry.* TO 'sentry-user'@'%' IDENTIFIED BY 'somepassword';

CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON oozie.* TO 'oozie-user'@'%' IDENTIFIED BY 'somepassword';

FLUSH PRIVILEGES;
SHOW DATABASES;
EXIT;

** Setup the CM database
sudo /usr/share/cmf/schema/scm_prepare_database.sh mysql scm scm-user somepassword
sudo rm /etc/cloudera-scm-server/db.mgmt.properties
sudo systemctl start cloudera-scm-server

GRANT ALL ON *.* TO 'training'@'%' IDENTIFIED BY 'training';



