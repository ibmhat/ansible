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

## uri

```
- name: Check that you can connect (GET) to a page and it returns a status 200
  uri:
    url: http://www.example.com

ansible inventory -m uri -a "url=http://www.example.com"
```

## url

```
- name: Download foo.conf
  get_url:
    url: http://example.com/path/file.conf
    dest: /etc/foo.conf
    mode: '0440'
   
```

## raw

- Executes a low-down and dirty SSH command, not going through the module subsystem.
- This is useful and should only be done in a few cases. A common case is installing python on a system without python installed by default. Another is speaking to any devices such as routers that do not have any Python installed. In any other case, using the shell or command module is much more appropriate.
- Arguments given to raw are run directly through the configured remote shell.
- Standard output, error output and return code are returned when available.
- There is no change handler support for this module.
- This module does not require python on the remote system, much like the script module.
- This module is also supported for Windows targets.

```
- name: Bootstrap a host without python2 installed
  raw: dnf install -y python2 python2-dnf libselinux-python

- name: Run a command that uses non-posix shell-isms (in this example /bin/sh doesn't handle redirection and wildcards together but bash does)
  raw: cat < /tmp/*txt
  args:
    executable: /bin/bash

- name: Safely use templated variables. Always use quote filter to avoid injection issues.
  raw: "{{ package_mgr|quote }} {{ pkg_flags|quote }} install {{ python|quote }}"

- name: List user accounts on a Windows system
  raw: Get-WmiObject -Class Win32_UserAccount
```

## script – Runs a local script on a remote node after transferring it

