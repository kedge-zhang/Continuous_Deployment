# Continuous integration 
# PHP
## 1.Utility：tomcat + jenknins + gitlab + git + jdk
## 2. Install utility
### 2.1 jdk1.8
```
##1.vim /etc/profile
export JAVA_HOME=/usr/java
export JRE_HOME=/usr/java/jre
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
##2.source /etc/profile
```
### 2.2 git2.13
```
yum install asciidoc xmlto curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker -y

cd /usr/local/src && wget https://www.kernel.org/pub/software/scm/git/git-2.13.0.tar.gz --no-check-certificate
 
tar fvx git-2.13.0.tar.gz && cd git-2.13.0 && make configure && ./configure && make all doc && make install install-doc install-html

mv /usr/local/src/git-2.13.0 /usr/local/git2.13
```
#### 2.2.1 git environment
```
vim /etc/profile
export PATH=/usr/local/git-2.13.0/bin:$PATH  
```
或者
```
whereis git
rm -rf  /usr/bin/git /usr/local/bin/git
ln -s /usr/local/git2.13/git /usr/bin/git
ln -s /usr/local/git2.13/git  /usr/local/bin/git
```
### 2.3 tomcat8 + jenknins
```
##1.tomcat
cd  /usr/local/src && wget http://mirror.bit.edu.cn/apache/tomcat/tomcat-8/v8.5.23/bin/apache-tomcat-8.5.23.tar.gz 

tar fvx apache-tomcat-8.5.23.tar.gz -C /data &&  mv /data/apache-tomcat-8.5.23 /data/tomcat8

```

```
##1.vim  /data/tomcat8/bin/catalina.sh 
JAVA_OPTS= '-Xms1024M -Xmx2048M -XX:PermSize=512M -XX:MaxNewSize=512M -XX:MaxPermSize=512M'
export CATALINA_OPTS="-DJENKINS_HOME=/data/jenkins"
export JENKINS_JAVA_OPTIONS="-Djava.awt.headless=true -Dhudson.ClassicPluginStrategy.noBytecodeTransformer=true"

##2.vim  /data/tomcat8/conf/server.xml
<Connector port="8080" protocol="HTTP/1.1"
               maxThreads="200"
               minSpareThreads="25"
               maxSpareThreads="75"
               acceptCount="50"
               debug="0"
               connectionTimeout="20000"
               redirectPort="8443"
               URIEncoding="UTF-8" />
```

```
mkdir -p /data/jenkins
cd /data/tomcat8/webapps && rm -rf ./* 
wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war
/data/tomcat8/bin/catalina.sh start && tail -f /data/tomcat8/logs/catalina.out
```
## 2.4 gitlab
[Gitlab Install https://about.gitlab.com/installation](https://about.gitlab.com/installation)
### 2.4.1centos6
```
yum install -y curl policycoreutils-python openssh-server cronie
lokkit -s http -s ssh
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash
EXTERNAL_URL="http://120.92.104.176" yum -y install gitlab-ee
```
### 2.4.2centos7
```
yum install -y curl policycorecurl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bashutils-python openssh-server
systemctl enable sshd
systemctl start sshd
firewall-cmd --permanent --add-service=http
systemctl reload firewalld
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash
sudo EXTERNAL_URL="http://120.92.104.176" yum install -y gitlab-ee
```
### 2.4.3 清华镜像
```
(cat << EOF
[gitlab-ce]
name=Gitlab CE Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/
gpgcheck=0
enabled=1
EOF
) > /etc/yum.repos.d/gitlab-ce.repo

yum makecache
yum install gitlab-ce

#Ps:此种方法安装修改比较麻烦，不建议此种操作
```

### 2.4.4gitlab组成
```
GitLab由以下服务构成：

nginx：静态Web服务器
gitlab-shell：用于处理Git命令和修改authorized keys列表
gitlab-workhorse:轻量级的反向代理服务器
logrotate：日志文件管理工具
postgresql：数据库
redis：缓存数据库
sidekiq：用于在后台执行队列任务（异步执行）
unicorn：An HTTP server for Rack applications，GitLab Rails应用是托管在这个服务器上面的。
```
### 2.4.5这里涉及到问题

1.8080端口占用
```
vim /etc/gitlab/gitlab.rb
```
![](https://i.imgur.com/D39aZjn.png)

```
 vim /var/opt/gitlab/gitlab-rails/etc/unicorn.rb
```
![](https://i.imgur.com/pQSFLt8.png)
```
vim /var/opt/gitlab/gitlab-shell/config.yml
```
![](https://i.imgur.com/nufVbtr.png)

2.修改gitlab仓库位置,关键字：git_data_dir

![git_data_dir](https://i.imgur.com/kQkw0Q3.png)

3.gitlab项目链接
```
vim /opt/gitlab/embedded/service/gitlab-rails/config/gitlab.yml
```
![](https://i.imgur.com/awdXSrK.png)

4.汉化
```
>1.克隆汉化包并指定匹配gitlab的版本
cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
head -1 /opt/gitlab/version-manifest.txt
cd /usr/local/src &&  git clone https://gitlab.com/xhang/gitlab.git -b v10.1.0-zh
cat gitlab/VERSION
>2.比较汉化标签和原标签，导出 patch 用的 diff 文件到/root下 
cd /usr/local/src/gitlab &&  git diff v10.1.0 v10.1.0-zh > /root/10.1.0-zh.diff
>3.将10.0.1-zh.diff作为补丁更新到gitlab中 
yum install patch -y
patch -d /opt/gitlab/embedded/service/gitlab-rails -p1 < 10.1.0-zh.diff
```
### 2.4.5 卸载
```
sudo gitlab-ctl stop
sudo gitlab-ctl uninstall
sudo gitlab-ctl cleanse
sudo rm -rf /opt/gitlab
yum remove gitlab-ee -
```

## 3.jenkins + gitlab
### 我的客户端IP： 120.92.78.208:22122
### 3.1 安装插件
![](https://i.imgur.com/qrK4VZV.png)
![](https://i.imgur.com/n0wfNho.png)
![](https://i.imgur.com/dUiPX5G.png)
![](https://i.imgur.com/Cteh4H0.png)
![](https://i.imgur.com/Lqvv2K6.png)
![](https://i.imgur.com/BCZRjQv.png)
![](https://i.imgur.com/wL7QsTG.png)
![](https://i.imgur.com/TNlUccD.png)
![](https://i.imgur.com/K621w3z.png)

### 3.2 Global Tool Configuration
![](https://i.imgur.com/ixysPPD.png)

### 3.3 管理节点和Credentials
在client（项目服务器）上生成密钥，然后复制id_rsa内容复制到这里
![](https://i.imgur.com/YS821VM.png)

![](https://i.imgur.com/rA9NFhY.png)

![](https://i.imgur.com/0gqqdap.png)

### 3.4 新建项目
选择“构建一个自由风格的软件项目”，然后“OK”