## cdh admin 시 distribution 단계가 진행되지 않을 때
- /etc/yum.repos.d/cloudera-manager.repo 상의 cdh version 을 신버젼으로 update

```
# 5.7 -> 5.11
baseurl=https://archive.cloudera.com/cm5/redhat/6/x86_64/cm/5.15.2/
```

- 재 인스톨 후 stop/start

```
sudo yum install -y cloudera-manager-daemons cloudera-manager-server

sudo systemctl stop cloudera-scm-server
sudo systemctl start cloudera-scm-server
```
