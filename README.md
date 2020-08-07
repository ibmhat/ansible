# ansible
ansible playbook

## ansible become

ansible xx -m ping --become --become-method=su --become-user=

ansible needbecome -m lineinfile -a "dest=/etc/rsyslog.conf line='*.*    @13.16.20.5:514'" --become --become-method=su --become-user=root 

添加参数不用加 --ask-become-pass  --ask-su-pass

`ansible_become_pass = "rootpasswd"`

ansible test -m shell -a "cd /etc/yum.repos.d && mkdir bak && mv *.repo bak"

ansible test -m copy-a "src=template/local.repo dest=/etc/yum.repos.d/local.repo"

## ansible when

```
        when: (ansible_facts['distribution'] == "CentOS" and ansible_facts['distribution_major_version'] == "6") or
              (ansible_facts['distribution'] == "Debian" and ansible_facts['distribution_major_version'] == "7")
```

```
---
- hosts: test
  tasks:
  - name: copy repo7
    copy: src=template/local7.repo dest=/etc/yum.repos.d/local.repo
    when: ansible_facts['distribution'] == 'CentOS' and ansible_facts['distribution_major_version'] == '7'
  - name: copy repo6
    copy: src=template/local6.repo dest=/etc/yum.repos.d/local.repo
    when: ansible_facts['distribution'] == 'CentOS' and ansible_facts['distribution_major_version'] == '6'
```

## ansible issue

**Aborting, target uses selinux but python bindings (libselinux-python) aren't installed!**

[参考这里](http://blog.leanote.com/post/benmo/ansible连接客户端selinux问题)

```
没有网络不能安装libselinux-python,可以通过修改ansible源代码来解决

/usr/lib/python2.6/site-packages/ansible/module_utils/basic.py

    def selinux_enabled(self):
        if not HAVE_SELINUX:
            # seenabled = self.get_bin_path('selinuxenabled')
            seenabled = self.get_bin_path('getenforce') #selinuxenabled 改为 getenforce
            if seenabled is not None:
                (rc, out, err) = self.run_command(seenabled)
                # if rc == 0:
                # if out not in ['Disabled','Permissive']: 
                if out == 'Enforcing':
                    self.fail_json(msg="Aborting, target uses selinux but python bindings (libselinux-python) aren't installed!")
            return False
        if selinux.is_selinux_enabled() == 1:
            return True
        else:
            return False

```

