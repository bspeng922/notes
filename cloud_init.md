User configurability
Cloud-init ‘s behavior can be configured via user-data.

User-data can be given by the user at instance launch time.
This is done via the --user-data or --user-data-file argument to ec2-run-instances for example.


Versions for other systems can be (or have been) created for the following distributions:
Ubuntu
Fedora
Debian
RHEL
CentOS


User-Data Script
Typically used by those who just want to execute a shell script.

Begins with: #! or Content-Type: text/x-shellscript when using a MIME archive.

Example
$ cat myscript.sh

#!/bin/sh
echo "Hello World.  The time is now $(date -R)!" | tee /root/output.txt




#!/bin/sh
passwd ubuntu<<EOF
ubuntu
ubuntu
EOF
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
service ssh restart