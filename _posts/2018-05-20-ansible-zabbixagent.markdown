---
layout: post
category: "zabbix"
title:  "用ansible批量安装zabbix-agent"
---

## 安装ansible
安装  
pip3 install ansible  
创建配置文件  
mkdir /etc/ansible  
vim /etc/ansible/ansible.cfg  
[ansible.cfg](https://raw.githubusercontent.com/ansible/ansible/devel/examples/ansible.cfg)

<!-- more -->

## 配置
创建hosts文件  
vim /etc/ansible/hosts
```
ip1
ip2
ip3
ip4
```

创建playbook  
vim /etc/ansible/zabbix_agent.yaml
```
\-\-\-
- hosts: all
  remote_user: root
  tasks:
  - name: copy zabbix-agent package
    copy: src=/etc/ansible/zabbix-agent-3.0.17-1.el6.x86_64.rpm dest=/tmp/zabbix-agent-3.0.17-1.el6.x86_64.rpm owner=root group=root mode=0755
  - name: install zabbix-agent package
    shell: rpm -ivh /tmp/zabbix-agent-3.0.17-1.el6.x86_64.rpm
  - name: copy zabbix-agent conf
    template: src=/etc/ansible/zabbix_agentd.conf dest=/etc/zabbix/zabbix_agentd.conf owner=root group=root mode=0644
    notify:
      - Restart Zabbix-agent Service
  handlers:
  - name: Restart Zabbix-agent Service
    service: name=zabbix-agent state=restarted
```

放置本地文件  
copy zabbix-agent-3.0.17-1.el6.x86_64.rpm至/etc/ansible/  
copy zabbix_agentd.conf至/etc/ansible/  
编辑zabbix_agentd.conf  
vim /etc/ansible/zabbix_agentd.conf  

```
### Option: Hostname
#       Unique, case sensitive hostname.
#       Required for active checks and must match hostname as configured on the server.
#       Value is acquired from HostnameItem if undefined.
#
# Mandatory: no
# Default:
# Hostname=
Hostname={{ inventory_hostname }}
```  
其他参数按实际需求修改

运行playbook  
ansible-playbook zabbix_agent.yaml
```
PLAY [all] 
TASK [Gathering Facts] 
ok: [192.168.0.192]
TASK [copy zabbix-agent package]
changed: [192.168.0.192]
TASK [install zabbix-agent package]
 [WARNING]: Consider using the yum, dnf or zypper module rather than running rpm.  If you need to use command because yum, dnf or zypper is
insufficient you can add warn=False to this command task or set command_warnings=False in ansible.cfg to get rid of this message.
changed: [192.168.0.192]
TASK [copy zabbix-agent conf]
changed: [192.168.0.192]
RUNNING HANDLER [Restart Zabbix-agent Service] 
changed: [192.168.0.192]
PLAY RECAP 
192.168.0.192              : ok=5    changed=4    unreachable=0    failed=0   
```