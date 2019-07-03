```
1. 로컬 bash_profile alais 세팅
2. 로컬 centos@퍼블릭 ip host 파일 세팅
3. 각 서버에서 host 파일 세팅
```

```
alias h1='ssh -i ./SKT.pem centos@15.164.189.253'
alias h2='ssh -i ./SKT.pem centos@15.164.189.57'
alias h3='ssh -i ./SKT.pem centos@15.164.29.241'
alias h4='ssh -i ./SKT.pem centos@15.164.29.93'
alias h5='ssh -i ./SKT.pem centos@15.164.85.137'

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
```

## 8. Check to see that ntp service is running, Disable chrony as necessary
```
# systemctl  list-units --type service -all | grep ntp
```
- 설치 되어있지 않으면 
```
# yum install ntp
# service ntpd start 
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

## cm server 설치
```
yum install cloudera-manager-daemons cloudera-manager-server
```
## cm agent 설치
```
sudo yum install cloudera-manager-daemons cloudera-manager-agent
```
