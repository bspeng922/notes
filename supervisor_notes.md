# supervisor 

## install

```

pip install supervisor

mkdir -p /var/log/supervisord/

echo_supervisord_conf > /etc/supervisord.conf

```


## 添加自定义配置文件路径

```

curr_dir=`pwd`
cat <<EOF >> /etc/supervisord.conf
[include]
files = $curr_dir/conf/*.ini
EOF

```


## 配置supervisor开机启动

```
cat <<EOF >> /usr/lib/systemd/system/supervisord.service
# dservice for systemd (CentOS 7.0+)
[Unit]
Description=Supervisor daemon

[Service]
Type=forking
ExecStart=/usr/bin/supervisord -c /etc/supervisord.conf
ExecStop=/usr/bin/supervisorctl shutdown
ExecReload=/usr/bin/supervisorctl reload
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
EOF

systemctl enable supervisord.service
systemctl start supervisord.service

```

## 自定义的配置

```

[program:file_watcher]
command=/usr/bin/python2 file_watcher.py
numprocs=1
process_name=%(program_name)s
directory=/root/gamebox
user=root
autostart=true
autorestart=true
startsecs=10
startretries=5
redirect_stderr=true
stdout_logfile=/var/log/supervisord/file_watcher.log
loglevel=info

```




