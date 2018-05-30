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
配置ssh密钥  
ssh-keygen  
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.0.xx

创建hosts文件  
vim /etc/ansible/hosts
```
[webservers]
192.168.0.xx
ip2
ip3
ip4
```

创建playbook  
vim /etc/ansible/zabbix_agent.yaml
```
\---
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
Hostname=\{\{ inventory_hostname \}\}
```  
其他参数按实际需求修改

运行playbook  
ansible-playbook zabbix_agent.yaml
```
PLAY [all] 
TASK [Gathering Facts] 
ok: [192.168.0.xx]
TASK [copy zabbix-agent package]
changed: [192.168.0.xx]
TASK [install zabbix-agent package]
 [WARNING]: Consider using the yum, dnf or zypper module rather than running rpm.  If you need to use command because yum, dnf or zypper is
insufficient you can add warn=False to this command task or set command_warnings=False in ansible.cfg to get rid of this message.
changed: [192.168.0.xx]
TASK [copy zabbix-agent conf]
changed: [192.168.0.xx]
RUNNING HANDLER [Restart Zabbix-agent Service] 
changed: [192.168.0.xx]
PLAY RECAP 
192.168.0.xx              : ok=5    changed=4    unreachable=0    failed=0   
```

如果不愿意搞ssh互信的话，密码可以直接写在hosts文件里  
先安装sshpass  
yum install -y sshpass  
再修改hosts文件  
vim /etc/ansible/hosts
```
[webservers]
192.168.0.xx ansible_ssh_pass='passwd'
ip2 ansible_ssh_pass='passwd'
ip3 ansible_ssh_pass='passwd'
ip4 ansible_ssh_pass='passwd'
```

另外补一个zabbix常用的playbook，修改/var/log/messages权限
vim /etc/ansible/chmod_syslog.yaml  
```
\---
- hosts: all
  remote_user: root
  tasks:
  - name: change syslog mode
    file: dest=/var/log/messages owner=root group=root mode=0644
  - name: change logrotate
    lineinfile:
      dest: /etc/logrotate.d/syslog
      regexp: '^endscript'
      insertbefore: '^}'
      line: '    create 0644 root root'
```
