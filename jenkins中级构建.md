# jenkins + salt
## 1.salt
### 1.1部署salt
```
yum -y install salt-master
yum -y install salt-minion
```
### 1.2master端口
```
    4505：提供远程执行命令发送功能
    4506：支持认证，文件服务，结果收集等
```
### 1.3 salt minion配置
```
vim  /etc/salt/minion 
  #master: salt---->master: salt
  #id: ---->id: minion-one
```
### 1.4 启动
```
service salt-master start
service salt-minion start
```
### 签发证书（密钥）
```
##master上执行：salt-key -L

##验证指纹码确保密钥匹配
master:salt-key -f  minion-one
minion：salt-call --local key.finger

##接受密钥：salt-key -a minion-one
```

### 2.INSTALL

```
#!/bin/bash

##date :20171111
##author:kedge_zhang

############################ 变量 ##############################
# 项目名称与分支引用jenkins
# 代码目录
CODE_PATH="/data/jenkins/workspace/$JOB_NAME"
# minon 项目目录
DEMO_PATH="/data"
#代码打包后存放目录
PACKAGE_PATH="/srv/salt/projects/$JOB_NAME/package"
#配置文件目录
CONFIG_PATH="/srv/salt/projects/$JOB_NAME/config"
#日志
LOG_FILE="/srv/salt/projects/$JOB_NAME/deploy.log"
#salt代码目录
SALT_PATH="projects/$JOB_NAME/package"
#日期
CTIME=$(date "+%Y%m%d-%H-%M")
#pull 到本地代码的MD5
############################## project_dir ########################
if [ ! -d $PACKAGE_PATH ];then
   mkdir -p "$PACKAGE_PATH"
   echo "$PACKAGE_PATH 是新建目录"
else 
   echo "$PACKAGE_PATH 存在" > /dev/null
fi

if [ -d $CONFIG_PATH ];then
   echo  "目录存在" > /dev/null
else
   echo "配置文件不存在"
   exit 1
fi

############################## tarfile ###########################
#[ ! -d $CODE_PATH ]  && echo "Iteam  not exist " && exit 0
cp -rf $CODE_PATH $PACKAGE_PATH
cp -rf $CONFIG_PATH/*  $PACKAGE_PATH/$JOB_NAME
cd $PACKAGE_PATH && tar zcfv $JOB_NAME.tar.gz $JOB_NAME

############################## INSTALL ############################
case $test_one in
dev)
salt "minion-one" cp.get_file salt://$SALT_PATH/$JOB_NAME.tar.gz $DEMO_PATH/$JOB_NAME.tar.gz
salt "minion-one" cmd.run "cd $DEMO_PATH && mv $JOB_NAME .backup/$JOB_NAME.$CTIME && tar  fvx $JOB_NAME.tar.gz && rm -rf $JOB_NAME.tar.gz"
rm -rf $PACKAGE_PATH/$JOB_NAME && rm -rf $PACKAGE_PATH/$JOB_NAME.tar.gz
;;
test)
salt "minion-" cp.get_file salt://$SALT_PATH/$JOB_NAME.tar.gz $DEMO_PATH/$JOB_NAME.tar.gz
salt "minion-" cmd.run "cd $DEMO_PATH && mv $JOB_NAME .backup/$JOB_NAME.$CTIME && tar  fvx $JOB_NAME.tar.gz && rm -rf $JOB_NAME.tar.gz"
rm -rf $PACKAGE_PATH/$JOB_NAME && rm -rf $PACKAGE_PATH/$JOB_NAME.tar.gz
;;
master)
salt "minion-localhost" cp.get_file salt://$SALT_PATH/$JOB_NAME.tar.gz $DEMO_PATH/$JOB_NAME.tar.gz
salt "minion-localhost" cmd.run "cd $DEMO_PATH && mv $JOB_NAME .backup/$JOB_NAME.$CTIME && tar  fvx $JOB_NAME.tar.gz && rm -rf $JOB_NAME.tar.gz"
rm -rf $PACKAGE_PATH/$JOB_NAME && rm -rf $PACKAGE_PATH/$JOB_NAME.tar.gz
;;
esac
```