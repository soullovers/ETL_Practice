1. 로컬 bash_profile alais 세팅

2. 로컬 centos@퍼블릭 ip host 파일 세팅

3. 각 서버에서 host 파일 세팅

4. Update yum
```
sudo yum update
```
5. Change the run level to multi-user text mode
```
sudo systemctl get-default
sudo systemctl set-default mulit-user.target
```
6. Disable SE Linux 
```
vi /etc/sysconfig/selinux
SELINUX=enforcing -> SELINUX=disabled
```
로 변경후 저장한다.
reboot

7. Disable firewall 
```
```