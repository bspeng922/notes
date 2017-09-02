# cinder-backup驱动配置


后端采用Swift

```
backup_swift_url = http://controller:8080/v1/AUTH_
backup_swift_auth_url = http://controller:5000/v3
swift_catalog_info = object-store:swift:publicURL
backup_swift_auth = per_user
backup_swift_auth_version = 3
backup_swift_container = volumebackups
backup_swift_object_size = 52428800
backup_swift_block_size = 32768
backup_swift_retry_attempts = 3
backup_swift_retry_backoff = 2
backup_swift_enable_progress_timer = true
backup_driver = cinder.backup.drivers.swift
```


重启cinder-volume、cinder-backup服务

```
systemctl restart openstack-cinder-volume  openstack-cinder-backup
```


horizon启动cinder backup功能，配置/etc/openstack-dashboard/local_settings

```
OPENSTACK_CINDER_FEATURES = {
    'enable_backup': True,
}
```


