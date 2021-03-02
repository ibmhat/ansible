## ansible 加密密码
``` 
ansible all -i localhost, -m debug -a "msg={{ 'mypassword' | password_hash('sha512', 'mysecretsalt') }}"
mkpasswd --method=sha-512
pip3 install passlib
python3 -c "from passlib.hash import sha512_crypt; import getpass; print(sha512_crypt.using(rounds=5000).hash(getpass.getpass()))"
```

## ansible 批量添加账号

```
#密码出问题，请用playbook
ansible all --become --become-user root --become-method su -m shell -a "/usr/sbin/useradd yourname && echo 'yourpasswd' |passwd --stdin yourname"

ansible all --become --become-user root --become-method su -m shell -a "echo 'yourname         ALL=(ALL)       NOPASSWD: ALL' >> /etc/sudoers"

在主机列表里面可以添加
ansible_become_pass = "rootpasswd"
```

## 修改zabbix Hostname

```
ansible all -m lineinfile -a "path='/usr/local/zabbix/etc/zabbix_agentd.conf' \
    regexp='^Hostname=' \
    line='Hostname={{ inventory_hostname }}'" -b --become-method=su --become-user=root
# apache-tomcat-8.5.63 会复制到tomcat_api 下
ansible epg_api -m copy -a "src=/web/soft/apache-tomcat-8.5.63 dest=/web/soft/tomcat_api remote_src=yes"
# apache-tomcat-8.5.63下的内容会复制到tomcat_cms_api 下
ansible epg_api -m copy -a "src=/web/soft/apache-tomcat-8.5.63/ dest=/web/soft/tomcat_cms_api/ remote_src=yes"

ansible all -m shell -a "cd /web/soft/Agent/AssetAgent_64_linux/ && nohup ./AssetAgent_start.sh &" --become --become-method su
ansible all -m unarchive -a "src=/web/soft/Agent/AssetAgent_64_linux.zip dest=/web/soft/Agent/ mode=0755" --become --become-method su
```

