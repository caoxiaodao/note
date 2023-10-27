# 常用命令

## 防火墙

+ 规规查看
  
  ```
  firewall-cmd --zone=public --list-all
  ```
- 防火墙规则重载
  
  ```firewall-cmd
  firewall-cmd --reload
  ```

- 防火墙向指定ip开放
  
  ```
  firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="10.44.237.102"   accept"
  ```

- 防火墙指定ip和端口开发
  
  ```
  firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.44.237.102" port protocol="tcp" port="31600-31800" accept'
  ```

## 网络配置和管理

### 修改网卡配置

常看网络连接名字 nmcli connection

配置名为system eth0 的网络接口的静态ip地址为10.58.90.34/24

    nmcli connection modify "System eth0" ipv4.addresses 10.58.90.34/24

修改设置主机名

查看主机名hostname

修改主机名hostnamectl set-hostname myserver.example.com

域名解析

vim /etx/hosts   192.168.1.200 www.example.com 

### 