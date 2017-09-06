# gitlab和jenkins集成

------

## 安装gitlab

1.安装相关依赖
```
yum install curl openssh-server openssh-clients postfix cronie -y

//如果要使用邮件功能，需要开启postfix服务
service postfix start

//在CentOS中，需要防火墙开启允许HTTP和SSH的访问
lokkit -s http -s ssh
```

2.安装gitlab
```
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash

yum install gitlab-ce -y



[sonar和jenkins集成](https://www.ibm.com/developerworks/cn/devops/1612_qusm_jenkins/index.html)

