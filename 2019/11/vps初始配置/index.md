# vps初始配置



## 添加用户并授予root权限

```bash
adduser ubuntu
#记住自己输入了啥密码
usermod -g sudo ubuntu
```
<!--more-->

## 修改主机名

```bash
nano /etc/hostname
#改为想要的
nano /etc/hosts
#第一行改为127.0.0.1      localhost newname
hostname newname

```
重启，祈祷有用

## 修改防火墙

1.查看状态
```bash
sudo ufw status
```
2.启用
```bash
sudo ufw enable
```
3.开启/禁用
```bash
sudo ufw allow|deny [service]
#例：sudo ufw allow 22/tcp
```
4.删除规则
```bash
sudo ufw delete allow 53
```
记得开启防火墙前打开ssh端口！！！！

## 设置ssh密钥
```bash
#生成密钥
ssh-keygen
#下载id_rsa密钥
#把生成的公钥导入到ssh的配置文件中
cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys 
#设置访问权限，不正确会报错
chmod 600 authorized_keys
chmod 700 /root/.ssh

#打开ssh配置文件
nano /etc/ssh/sshd_config 
找到PubkeyAuthentication（在第37行）,默认的话，是被注释的，并且为no，我们把注释去掉，并且改为yes //开启密钥登陆
找到PasswordAuthentication（在第56行）,默认的话，是被注释的，并且为yes，我们把注释去掉，并且改为no //关闭密码登陆

#重启ssh服务
service sshd restart 
```