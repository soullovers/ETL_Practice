https://drive.google.com/drive/folders/1j5njKkpqaGErnYhiH49t5rKGbVLINYpp
```
1. 로컬 bash_profile alais 세팅
2. 로컬 centos@퍼블릭 ip host 파일 세팅
3. 각 서버에서 host 파일 세팅
```

```
#public
15.164.168.239	cm.bdai.com	cm
15.164.200.129	m1.bdai.com	m1
15.164.204.53	d1.bdai.com	d1
15.164.77.16	d2.bdai.com	d2
52.78.111.92	d3.bdai.com	d3

#private
172.31.0.97	cm.bdai.com	cm
172.31.3.221	m1.bdai.com	m1
172.31.15.212	d1.bdai.com	d1
172.31.14.47	d2.bdai.com	d2
172.31.3.141	d3.bdai.com	d3

alias cms='ssh -i ./SKT.pem centos@15.164.168.239'
alias m1s='ssh -i ./SKT.pem centos@15.164.200.129'
alias d1s='ssh -i ./SKT.pem centos@15.164.204.53'
alias d2s='ssh -i ./SKT.pem centos@15.164.77.16'
alias d3s='ssh -i ./SKT.pem centos@52.78.111.92'
```

# host 간 ssh 연결 작업
- pem key 파일을 cm server host 에 복사
```
scp -i SKT.pem SKT.pem centos@15.164.29.241:.
chmod 400 ./SKT.pem
ssh -i SKT.pem centos@host1~5
(1회씩 접속하여 key change 수행)
```
- hostname 을 /etc/hosts 에 서로 참조되는 이름으로 변경
```
sudo hostnamectl set-hostname host3

reboot
```

# System Pre-configuration Checks
## 1. Update yum
```
sudo yum update
```
## 2. Change the run level to multi-user text mode
```
sudo systemctl get-default
sudo systemctl set-default multi-user.target
```
## 3. Disable SE Linux 
```
vi /etc/sysconfig/selinux
SELINUX=enforcing -> SELINUX=disabled
```
로 변경후 저장한다.
reboot

## 4. Disable firewall 
```
```

## 5. Check vm.swappiness and update permanently as necessary : Set the value to 1
```
# vi /etc/sysctl.conf

vm.swappiness = 1
```
(reboot 필요)

- 확인
```
sysctl vm.swappiness
```

## 6. Disable transparent hugepage support permanently
- `[always]` 라 표시되면 ENABLE 되어잇는 상태
- `[never]` 라 표시되면 성공
```
# cat /sys/kernel/mm/transparent_hugepage/enabled
[always] madvise never
```
- /etc/default/grub 파일의 GRUB_CMDLINE_LINUX 라인 파라미터에 transparent_hugepage=never 추가
```
# vi /etc/default/grub
GRUB_TIMEOUT=5
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="nomodeset crashkernel=auto rd.lvm.lv=vg_os/lv_root rd.lvm.lv=vg_os/lv_swap rhgb quiet transparent_hugepage=never"
GRUB_DISABLE_RECOVERY="true"

# grub2-mkconfig -o /boot/grub2/grub.cfg

(reboot 필요)

# cat /proc/cmdline
BOOT_IMAGE=/vmlinuz-3.10.0-514.10.2.el7.x86_64 root=/dev/mapper/vg_os-lv_root ro nomodeset crashkernel=auto rd.lvm.lv=vg_os/lv_root rd.lvm.lv=vg_os/lv_swap rhgb quiet transparent_hugepage=never LANG=en_US.UTF-8
```

## 7. Check to see that nscd service is running
```
# systemctl  list-units --type service -all | grep nscd
```
- 설치 되어있지 않으면 
```
# yum install nscd
# service nscd start 
# sudo systemctl enable nscd
```

## 8. Check to see that ntp service is running, Disable chrony as necessary
```
# systemctl  list-units --type service -all | grep ntp
```
- 설치 되어있지 않으면 
```
# yum install ntp
# service ntpd start 
# sudo systemctl enable ntpd
```

- chronyd 영구 disable
```
systemctl disable chronyd
service chronyd stop

(reboot 후 확인해볼것)
```

## 9. Disable ipv6

- /etc/default/grub 파일의 GRUB_CMDLINE_LINUX 라인 파라미터에 ipv6.disable=1 추가
```
# vi /etc/default/grub
GRUB_TIMEOUT=5
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="nomodeset crashkernel=auto rd.lvm.lv=vg_os/lv_root rd.lvm.lv=vg_os/lv_swap rhgb quiet transparent_hugepage=never ipv6.disable=1"
GRUB_DISABLE_RECOVERY="true"

# grub2-mkconfig -o /boot/grub2/grub.cfg

(reboot 필요)

ip addr show | grep net6
```

## 10. 각 개인PC -> instances 로 ssh 접속 1회씩 할것(key exchange)

## 11. Show that forward and reverse host lookups are correctly resolved
- 각 instance 의 /etc/hosts 에 호스트 추가
```
172.31.15.98	host1
172.31.1.63	host2
172.31.14.58	host3
172.31.9.201	host4
172.31.8.29	host5
```

# Path B install using CM 5.15.x
## 1. jdk 설치
```
sudo su
mv /home/centos/jdk-8u211-linux-x64.rpm /usr/local/
chmod 777 /usr/local/jdk-8u211-linux-x64.rpm
yum localinstall /usr/local/jdk-8u211-linux-x64.rpm
```

- 잘못 설치된 jdk 삭제시 정확한 패키지 이름 확인
```
# rpm -qa | grep java

javapackages-tools-3.4.1-11.el7.noarch
tzdata-java-2019a-1.el7.noarch
java-1.8.0-openjdk-headless-1.8.0.212.b04-0.el7_6.x86_64
python-javapackages-3.4.1-11.el7.noarch
java-1.8.0-openjdk-1.8.0.212.b04-0.el7_6.x86_64

```
- JAVA_HOME 설정 : http://wincloud.link/pages/viewpage.action?pageId=3080270

## 2. jdbc connector
- 내부의 jar 파일을 java lib 로 복사
```
wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.47.tar.gz
tar -zxvf mysql-connector-java-5.1.47.tar.gz
sudo mkdir -p /usr/share/java/
cd mysql-connector-java-5.1.47
sudo cp mysql-connector-java-5.1.47-bin.jar /usr/share/java/mysql-connector-java.jar
```
- 자바 환경변수 ㅅ세팅
```
cp mysql-connector-java-5.1.47-bin.jar /usr/java/jdk1.8.0_211-amd64/lib/

vi /etc/profile
export JAVA_HOME=/usr/java/jdk1.8.0_211-amd64/ 추가
export CLASSPATH=$JAVA_HOME/lib/

source /etc/profile
```

## Step 1: Configure a Repository for Cloudera Manager
```
 sudo wget https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/cloudera-manager.repo -P /etc/yum.repos.d/
 sudo rpm --import https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/RPM-GPG-KEY-cloudera
 ```

## cm server 설치(on host3)
```
yum install cloudera-manager-daemons cloudera-manager-server
```
## cm agent 설치(on all hosts)
```
sudo yum install cloudera-manager-daemons cloudera-manager-agent
```
##  configure the Cloudera Manager Agent(on all hosts)
```
#vi /etc/cloudera-scm-agent/config.ini
server_host=host3
```

## Start the Agents
```
sudo systemctl start cloudera-scm-agent
```

## Install the MySQL database.
```
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm
sudo yum update
sudo yum install mysql-server
sudo systemctl start mysqld
```

## Configuring and Starting the MySQL Server
### stop server
```
#sudo systemctl stop mysqld
```
### Update my.cnf
```
#vi /etc/my.cnf

[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
transaction-isolation = READ-COMMITTED
# Disabling symbolic-links is recommended to prevent assorted security risks;
# to do so, uncomment this line:
symbolic-links = 0

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

#In later versions of MySQL, if you enable the binary log and do not set
#a server_id, MySQL will not start. The server_id must be unique within
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
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

sql_mode=STRICT_ALL_TABLES

```

### Ensure the MySQL server starts 
```
sudo systemctl enable mysqld
```

### Start the MySQL server
```
sudo systemctl start mysqld
```


### Run /usr/bin/mysql_secure_installation to set the MySQL root password and other security-related settings
```
sudo /usr/bin/mysql_secure_installation
```
```
[...]
Enter current password for root (enter for none):
OK, successfully used password, moving on...
[...]
Set root password? [Y/n] Y
New password:
Re-enter new password:
Remove anonymous users? [Y/n] Y
[...]
Disallow root login remotely? [Y/n] N
[...]
Remove test database and access to it [Y/n] Y
[...]
Reload privilege tables now? [Y/n] Y
All done!
```


## Creating Databases for Cloudera Software

```
CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE hive DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE nav DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE navms DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;




GRANT ALL ON scm.* TO 'scm'@'%' IDENTIFIED BY 'scm';
GRANT ALL ON amon.* TO 'amon'@'%' IDENTIFIED BY 'amon';
GRANT ALL ON rman.* TO 'rman'@'%' IDENTIFIED BY 'rman';
GRANT ALL ON hue.* TO 'hue'@'%' IDENTIFIED BY 'hue';
GRANT ALL ON hive.* TO 'hive'@'%' IDENTIFIED BY 'hive';
GRANT ALL ON sentry.* TO 'sentry'@'%' IDENTIFIED BY 'sentry';
GRANT ALL ON nav.* TO 'nav'@'%' IDENTIFIED BY 'nav';
GRANT ALL ON navms.* TO 'navms'@'%' IDENTIFIED BY 'navms';
GRANT ALL ON oozie.* TO 'oozie'@'%' IDENTIFIED BY 'oozie';
```


## Set up the Cloudera Manager Database

```
/usr/share/cmf/schema/scm_prepare_database.sh mysql scm scm scm
```

## start cm server
```
sudo service cloudera-scm-server start
```

# CDH Cluster Hosts and Role Assignments
### Master Hosts [host1]
- NameNode
- YARN ResourceManager
- JobHistory Server
- ZooKeeper
- Kudu master
- Spark History Server


### Utility Hosts, Gateway Hosts [host3]
- Secondary NameNode
- Cloudera Manager
- Cloudera Manager Management Service
- Hive Metastore
- HiveServer2
- Impala Catalog Server
- Impala StateStore
- Hue
- Oozie
- Flume
- Gateway configuration

### Worker Hosts [host2,host4,host5]
- DataNode
- NodeManager
- Impalad
- Kudu tablet server

## add hdfs user

```
sudo adduser training (이거슨 모든 노드에서 생성)
hdfs dfs -mkdir /user/training
hdfs dfs -chmown training:training /user/training
hdfs dfs -chmod 755 /user/training

```


```
sqoop import \
--connect jdbc:mysql://cm:3306/test \
--username training \
--password training \
--target-dir /user/training/authors \
--fields-terminated-by '\001' \
--split-by id \
--query "select id, first_name, last_name, email, birthdate, added from test.authors WHERE \$CONDITIONS"

sqoop import \
--connect jdbc:mysql://cm:3306/test \
--username training \
--password training \
--target-dir /user/training/posts \
--fields-terminated-by '\001' \
--split-by id \
--query "select * from test.posts WHERE \$CONDITIONS"
```

```
create database test;

create external table if not exists test.authors (
     id int ,
     first_name varchar(50),
     last_name varchar(50),
     email varchar(100),
     birthdate string,
     added timestamp
) 
comment 'test author'
row format delimited fields terminated by '\001'
location '/user/training/authors'
;

create external table if not exists test.posts (
     id int ,
     author_id int,
     title varchar(255),
     description varchar(500),
     content string,
     date string
) 
comment 'test post'
row format delimited fields terminated by '\001'
location '/user/training/posts'
;
```
