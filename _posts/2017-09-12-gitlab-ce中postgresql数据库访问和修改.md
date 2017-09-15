# gitlab-ce中postgresql的访问和设置

------

之前使用官网的rpm包安装的gitlab-ce，其rpm包中已经集成了众多组件，如nginx，postgresql，ruby...

1. 使用rpm -ql gitlab-ce来查看相应的软件包安装位置

2. 使用ps aux | grep postgresql,查看对应的运行程序，得到相应的postgresql的data目录(/var/opt/gitlab/postgresql),通过查看/etc/passwd，得到id为490其对应的用户为gitlab-psql

3. 切换到gitlab-psql用户，连接postgresql数据库 `psql`,直接连接报socket错误，指定对应的socket，并且要指定对应的数据库名 `psql -h /var/opt/gitlab/postgresql -d githubhq_production` 如果报该数据库不存在，可使用githubhq_pro尝试

4. 连接上数据库后，通过\dt查看该库中的表，找到对应的users表，修改对应id的encrypted_password的值。 `update users set encrypted_password='' where id=''`

5. 访问对应的页面进行登录


**默认的用户名: admin@example.com	密码: 5iveL!fe**
