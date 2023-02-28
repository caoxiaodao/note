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