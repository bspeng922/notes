# cloud init

## 介绍
User configurability Cloud-init ‘s behavior can be configured via user-data.

User-data can be given by the user at instance launch time.This is done via the --user-data or --user-data-file argument to ec2-run-instances for example.


## 使用

cloud init 使用的前提是镜像中已经安装了cloud-init 包，如果使用python代码（三个双引号）动态生成 user-data，需要注意的是行前一定不要留有空格，否则脚本不能执行！

### 执行shell脚本

Begins with: #! or Content-Type: text/x-shellscript when using a MIME archive.

```shell
$ cat myscript.sh

#!/bin/sh
echo "Hello World.  The time is now $(date -R)!" | tee /root/output.txt
```

修改ubuntu的默认密码，并开启ssh密码登录

```shell
#!/bin/sh
passwd ubuntu<<EOF
ubuntu
ubuntu
EOF
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
service ssh restart
```

### 执行cloud-config配置

```
#cloud-config
ssh_pwauth: true
disable_root: 0
user: root
password: 123456
chpasswd:
  expire: false
```

> 首先, 这个文件把一般云主机里面的 ssh 密码认证打开, 默认关闭的
> 然后, 通过 disable_root 允许 root 认证, 默认关闭
> 将 root 的密码设置为 abc123
> 如果不设置 chpasswd 的 expire 为 false, 那么登陆的时候会提示马上修改密码才能进去


### 同时传递多种格式/多个文件的 user-data

```shell
Content-Type: multipart/mixed; boundary="===============2197920354430400835=="
MIME-Version: 1.0

--===============2197920354430400835==
Content-Type: text/cloud-config; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="cloudconfig.txt"

#cloud-config
runcmd:
 - touch /root/cloudconfig
 - source: "ppa:smoser/ppa"

--===============2197920354430400835==
Content-Type: text/x-include-url; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="includefile.txt"

# these urls will be read pulled in if they were part of user-data
# comments are allowed.  The format is one url per line
http://www.ubuntu.com/robots.txt

--===============2197920354430400835==
Content-Type: text/cloud-boothook; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="boothook.txt"

#!/bin/sh
echo "Hello World!"

--===============2197920354430400835==
Content-Type: text/x-shellscript; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="userscript.txt"

#!/usr/bin/perl
print "This is a user script (rc.local)\n"

--===============2197920354430400835==--
```


### 无法获取控制台日志

如果无法获取到云主机日志，则需要修改grub文件配置

+ ubuntu 

```shell
# vim /etc/default/grub
GRUB_CMDLINE_LINUX="console=tty0 console=ttyS0,115200"

# update-grub           #ubuntu
```

+ centos

```shell
# vi /boot/grub2/grub.cfg
## append "console=tty0 console=ttyS0,115200" to "CentOS ... (Core)" line
```


### 修改云主机domain

默认虚拟机的domain是novadomain，如果要自定义domain名称，需要修改nova的配置文件

```shell
# vi /etc/nova/nova.conf
dhcp_domain=testdomain

# systemctl restart openstack-nova-api.service
```


### 再次执行userdata

We can however fool cloud-init by letting it think the machine did a fresh first boot. We need to remove the following two files:

/var/lib/cloud/instances/$UUID/boot-finished
/var/lib/cloud/instances/$UUID/sem/config_scripts_user

Execute the following command to run the cloud-init final module again:

cloud-init modules --mode final
The final module will execute our user_data script again. Before every new test run you need to remove the two files listed above.


## User and Group Management

```
#cloud-config
users:
  - first_user_parameter
    first_user_parameter

  - second_user_parameter
    second_user_parameter
    second_user_parameter
```

The following keys are available for definition:

name: The account username.
primary-group: The primary group of the user. By default, this will be a group created that matches the username. Any group specified here must already exist or must be created explicitly (we discuss this later in this section).
groups: Any supplementary groups can be listed here, separated by commas.
gecos: A field for supplementary info about the user.
shell: The shell that should be set for the user. If you do not set this, the very basic sh shell will be used.
expiredate: The date that the account should expire, in YYYY-MM-DD format.
sudo: The sudo string to use if you would like to define sudo privileges, without the username field.
lock-passwd: This is set to "True" by default. Set this to "False" to allow users to log in with a password.
passwd: A hashed password for the account.
ssh-authorized-keys: A list of complete SSH public keys that should be added to this user's authorized_keys file in their .ssh directory.
inactive: A boolean value that will set the account to inactive.
system: If "True", this account will be a system account with no home directory.
homedir: Used to override the default /home/<username>, which is otherwise created and set.
ssh-import-id: The SSH ID to import from LaunchPad.
selinux-user: This can be used to set the SELinux user that should be used for this account's login.
no-create-home: Set to "True" to avoid creating a /home/<username> directory for the user.
no-user-group: Set to "True" to avoid creating a group with the same name as the user.
no-log-init: Set to "True" to not initiate the user login databases.


```
#cloud-config
groups:
  - group1
  - group2: [user1, user2]
```


## Change Passwords for Existing Users

For user accounts that already exist (the root account is the most pertinent), a password can be suppled by using the chpasswd directive.

```
#cloud-config
chpasswd:
  list: |
    user1:password1
    user2:password2
    user3:password3
  expire: False
```

One thing to note is that you can set a password to "RANDOM" or "R", which will generate a random password and write it to /var/log/cloud-init-output.log. 

## Write Files to the Disk

In order to write files to the disk, you should use the write_files directive.

The only required keys in this array are path, which defines where to write the file, and content, which contains the data you would like the file to contain.

The available keys for configuring a write_files item are:

path: The absolute path to the location on the filesystem where the file should be written.

content: The content that should be placed in the file. For multi-line input, you should start a block by using a pipe character (|) on the "content" line, followed by an indented block containing the content. Binary files should include "!!binary" and a space prior to the pipe character.

owner: The user account and group that should be given ownership of the file. These should be given in the "username:group" format.

permissions: The octal permissions set that should be given for this file.

encoding: An optional encoding specification for the file. This can be "b64" for Base64 files, "gzip" for Gzip compressed files, or "gz+b64" for a combination. Leaving this out will use the default, conventional file type.

```
#cloud-config
write_files:
  - path: /test.txt
    content: |
      Here is a line.
      Another line is here.
```


## Update or Install Packages on the Server

To update the apt database on Debian-based distributions, you should set the package_update directive to "true". This is synonymous with calling apt-get update from the command line.

```
#cloud-config
package_update: false
```


If you wish to upgrade all of the packages on your server after it boots up for the first time, you can set the package_upgrade directive. This is akin to a apt-get upgrade executed manually.

```
#cloud-config
package_upgrade: true
```

To install additional packages, you can simply list the package names using the "packages" directive. Each list item should represent a package. Unlike the two commands above, this directive will function with either yum or apt managed distros.

```
#cloud-config
packages:
  - package_1
  - package_2
  - [package_3, version_num]
```

The "packages" directive will set apt_update to true, overriding any previous setting.


## Configure SSH Keys for User Accounts

You can manage SSH keys in the users directive, but you can also specify them in a dedicated ssh_authorized_keys section. These will be added to the first defined user's authorized_keys file.

```
#cloud-config
ssh_authorized_keys:
  - ssh_key_1
  - ssh_key_2
```

You can also generate the SSH server's private keys ahead of time and place them on the filesystem. This can be useful if you want to give your clients the information about this server beforehand, allowing it to trust the server as soon as it comes online.

```
#cloud-config
ssh_keys:
  rsa_private: |
    -----BEGIN RSA PRIVATE KEY-----
    your_rsa_private_key
    -----END RSA PRIVATE KEY-----

  rsa_public: your_rsa_public_key
```


## Configure resolv.conf to Use Specific DNS Servers

If you have configured your own DNS servers that you wish to use, you can manage your server's resolv.conf file by using the resolv_conf directive. This currently only works for RHEL-based distributions.

```
#cloud-config
manage-resolv-conf: true
resolv_conf:
  nameservers:
    - 'first_nameserver'
    - 'second_nameserver'
  searchdomains:
    - first.domain.com
    - second.domain.com
  domain: domain.com
  options:
    option1: value1
    option2: value2
    option3: value3
```

If you are using the resolv_conf directive, you must ensure that the manage-resolv-conf directive is also set to true. Not doing so will cause your settings to be ignored


## Run Arbitrary Commands for More Control

If none of the managed actions that cloud-config provides works for what you want to do, you can also run arbitrary commands. You can do this with the runcmd directive.

Any output will be written to standard out and to the /var/log/cloud-init-output.log file:

```
#cloud-config
runcmd:
  - [ sed, -i, -e, 's/here/there/g', some_file]
  - echo "modified some_file"
  - [cat, some_file]
```


## Shutdown or Reboot the Server

In some cases, you'll want to shutdown or reboot your server after executing the other items. You can do this by setting up the power_state directive.

```
#cloud-config
power_state:
  timeout: 120
  delay: "+5"
  message: Rebooting in five minutes. Please save your work.
  mode: reboot
```


# Example:

Our adjusted strategy will look something like this:

Set no password and provide no SSH keys for the root account through cloud-config (any SSH keys added though the DigitalOcean interface will still be added as usual)
Create a new user
Set no password for the new user account
Set up SSH access for the new user account
Give the new user password-less sudo privileges to make administrative changes.
(Optional) Change the port the SSH daemon listens on
(Optional) Restrict root SSH login (especially if you do not include SSH keys through the DigitalOcean interface)
(Optional) Explicitly permit our new user


```
#cloud-config
users:
  - name: demo
    ssh-authorized-keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCv60WjxoM39LgPDbiW7ne3gu18q0NIVv0RE6rDLNal1quXZ3nqAlANpl5qmhDQ+GS/sOtygSG4/9aiOA4vXO54k1mHWL2irjuB9XbXr00+44vSd2q/vtXdGXhdSMTf4/XK17fjKSG/9y3yD6nml6q9XgQxx9Vf/IkaKdlK0hbC1ds0+8h83PTb9dF3L7hf3Ch/ghvj5++tWJFdFeG+VI7EDuKNA4zL8C5FdYYWFA88YAmM8ndjA5qCjZXIIeZvZ/z9Kpy6DL0QZ8T3NsxRKapEU3nyiIuEAmn8fbnosWcsovw0IS1Hz6HsjYo4bu/gA82LWt3sdRUBZ/7ZsVD3ELip user@example.com
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    groups: sudo
    shell: /bin/bash
runcmd:
  - sed -i -e '/^Port/s/^.*$/Port 4444/' /etc/ssh/sshd_config
  - sed -i -e '/^PermitRootLogin/s/^.*$/PermitRootLogin no/' /etc/ssh/sshd_config
  - sed -i -e '$aAllowUsers demo' /etc/ssh/sshd_config
  - restart ssh
```


