# Magnum安装配置

官方文档： [https://docs.openstack.org/project-install-guide/container-infrastructure-management/draft/install-rdo.html](https://docs.openstack.org/project-install-guide/container-infrastructure-management/draft/install-rdo.html)

从源码安装Magnum [https://docs.openstack.org/magnum/latest/install/install-guide-from-source.html](https://docs.openstack.org/magnum/latest/install/install-guide-from-source.html)


----

## Usage

```
 $ docker pull samalba/hipache

 $ docker save samalba/hipache | glance image-create --is-public=True --container-format=docker --disk-format=raw --name samalba/hipache

 $ glance image-create --is-public=True --container-format=docker --disk-format=raw --name img_httpd < img_httpd.tar

 Ensure that the image name is the same as the image tar file name. The container format must be docker and the disk format must be raw.
```



----


## swarm-cluster创建超时

现象就是swarm-master一直显示在创建中，然后超过60分钟后，显示创建失败，heat显示的错误是Timeout

**解决方法：**
超时问题是因为虚拟机中解析不到controller对应的IP地址，需要讲keystone, heat, magnum 的 public endpoint改成 IP地址。



----


## swarm-manager service failed to start

创建swarm-master是，提示swarm-manager服务启动失败，登录到虚拟机内发现etcd，swarm-manager服务都启动失败，etcd启动失败的原因是 “open /etc/docker/server.crt: no such file or directory” ，正常的/etc/docker/下面应该有三个证书文件，但错误的虚拟机里只有一个。

**错误日志** : /var/log/message

```
2017-11-02 15:41:35.325 24030 WARNING magnum.common.keystone [req-1f4cec3b-d7a7-4143-825a-728cda9b1a7d - - - - -] Auth plugin and its options for service user must be provided in [keystone_auth] section. Using values from [keystone_authtoken] section is deprecated.: MissingRequiredOptions: Auth plugin requires parameters which were not given: auth_url
2017-11-02 15:41:48.275 24026 ERROR magnum.drivers.heat.driver [req-609a5a1b-14f1-4e78-b22c-c93c806c36d9 - - - - -] Cluster error, stack status: CREATE_FAILED, stack_id: 420d9818-06a6-4a9c-9d86-f11aaf225eb2, reason: Resource CREATE failed: WaitConditionFailure: resources.swarm_masters.resources[0].resources.master_wait_condition: swarm-manager service failed to start.
2017-11-02 15:41:48.921 24029 ERROR magnum.drivers.heat.driver [req-9160df5d-338f-4acc-aab8-e3c8c3370703 - - - - -] Cluster error, stack status: CREATE_FAILED, stack_id: 420d9818-06a6-4a9c-9d86-f11aaf225eb2, reason: Resource CREATE failed: WaitConditionFailure: resources.swarm_masters.resources[0].resources.master_wait_condition: swarm-manager service failed to start.
```

**虚拟机服务状态** 

```
[fedora@swarm-cluster-i7bkbockqyvu-master-0 ~]$ sudo systemctl start swarm-manager
A dependency job for swarm-manager.service failed. See 'journalctl -xe' for details.
[fedora@swarm-cluster-i7bkbockqyvu-master-0 ~]$ journalctl -xe
Nov 01 06:55:44 swarm-cluster-i7bkbockqyvu-master-0.novalocal etcd[2452]: recognized and used environment variable ETCD_PEER_CERT_FILE=/etc/docker/server.crt
Nov 01 06:55:44 swarm-cluster-i7bkbockqyvu-master-0.novalocal etcd[2452]: recognized and used environment variable ETCD_PEER_KEY_FILE=/etc/docker/server.key
Nov 01 06:55:44 swarm-cluster-i7bkbockqyvu-master-0.novalocal etcd[2452]: recognized environment variable ETCD_NAME, but unused: shadowed by corresponding flag 
Nov 01 06:55:44 swarm-cluster-i7bkbockqyvu-master-0.novalocal etcd[2452]: recognized environment variable ETCD_DATA_DIR, but unused: shadowed by corresponding fl
Nov 01 06:55:44 swarm-cluster-i7bkbockqyvu-master-0.novalocal etcd[2452]: recognized environment variable ETCD_LISTEN_CLIENT_URLS, but unused: shadowed by corres
Nov 01 06:55:44 swarm-cluster-i7bkbockqyvu-master-0.novalocal etcd[2452]: etcd Version: 3.1.3
Nov 01 06:55:44 swarm-cluster-i7bkbockqyvu-master-0.novalocal etcd[2452]: Git SHA: 21fdcc6
Nov 01 06:55:44 swarm-cluster-i7bkbockqyvu-master-0.novalocal etcd[2452]: Go Version: go1.7.5
Nov 01 06:55:44 swarm-cluster-i7bkbockqyvu-master-0.novalocal etcd[2452]: Go OS/Arch: linux/amd64
Nov 01 06:55:44 swarm-cluster-i7bkbockqyvu-master-0.novalocal etcd[2452]: setting maximum number of CPUs to 4, total number of available CPUs is 4
Nov 01 06:55:44 swarm-cluster-i7bkbockqyvu-master-0.novalocal etcd[2452]: peerTLS: cert = /etc/docker/server.crt, key = /etc/docker/server.key, ca = /etc/docker/
Nov 01 06:55:44 swarm-cluster-i7bkbockqyvu-master-0.novalocal etcd[2452]: open /etc/docker/server.crt: no such file or directory
Nov 01 06:55:44 swarm-cluster-i7bkbockqyvu-master-0.novalocal systemd[1]: etcd.service: Main process exited, code=exited, status=1/FAILURE
Nov 01 06:55:44 swarm-cluster-i7bkbockqyvu-master-0.novalocal audit[1]: SERVICE_START pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0
Nov 01 06:55:44 swarm-cluster-i7bkbockqyvu-master-0.novalocal systemd[1]: Failed to start Etcd Server.
-- Subject: Unit etcd.service has failed
-- Defined-By: systemd
-- Support: http://lists.freedesktop.org/mailman/listinfo/systemd-devel
-- 
-- Unit etcd.service has failed.
-- 
-- The result is failed.
```

**Cloud-init 日志** ： /var/log/cloud-init-output.log

```
Cloud-init v. 0.7.9 running 'modules:config' at Thu, 02 Nov 2017 02:07:02 +0000. Up 39.38 seconds.
removing docker key
Traceback (most recent call last):
  File "/var/lib/cloud/instance/scripts/part-004", line 181, in <module>
    sys.exit(main())
  File "/var/lib/cloud/instance/scripts/part-004", line 174, in main
    write_ca_cert(config)
  File "/var/lib/cloud/instance/scripts/part-004", line 93, in write_ca_cert
    fp.write(ca_cert_resp.json()['pem'])
KeyError: 'pem'
Cloud-init v. 0.7.9 running 'modules:final' at Thu, 02 Nov 2017 02:07:05 +0000. Up 41.72 seconds.
2017-11-02 02:07:09,335 - util.py[WARNING]: Failed running /var/lib/cloud/instance/scripts/part-004 [1]
Configuring docker network ...

... ...

notifying heat
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   113  100     4  100   109      5    142 --:--:-- --:--:-- --:--:--   142
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 4
X-Openstack-Request-Id: req-96486cd4-c8ae-4465-b37b-31296c2383b9
  te: Thu, 02 Nov 2017 02:07:13 GMT
▽
null+ '[' -z '' ']'
+ exit 0
2017-11-02 02:07:14,039 - cc_scripts_user.py[WARNING]: Failed to run module scripts-user (scripts in /var/lib/cloud/instance/scripts)
2017-11-02 02:07:14,051 - util.py[WARNING]: Running module scripts-user (<module 'cloudinit.config.cc_scripts_user' from '/usr/lib/python3.5/site-packages/cloudinit/config/cc_scripts_user.py'>) failed
Cloud-init v. 0.7.9 finished at Thu, 02 Nov 2017 02:07:14 +0000. Datasource DataSourceOpenStack [net,ver=2].  Up 50.91 seconds

```

**解决方法**： 

**虚拟机初始化代码出错位置**

```
def write_server_cert(config, csr_req):
    cert_url = '%s/certificates' % config['MAGNUM_URL']
    headers = {
        'Content-Type': 'application/json',
        'X-Auth-Token': config['USER_TOKEN'],
        'OpenStack-API-Version': 'container-infra latest'
    }
    csr_resp = requests.post(cert_url,
                             data=json.dumps(csr_req),
                             headers=headers)

    with open(SERVER_CERT_PATH, 'w') as fp:
        fp.write(csr_resp.json()['pem'])
```

从出错的位置可以看出，就是虚拟机请求certificates时出了问题。在Postman中测试，先生成token，再请求certificates

### User Token

```
-- POST  http://10.10.80.10:5000/v3/auth/tokens

{
    "token": {
        "issued_at": "2017-11-01T08:44:05.000000Z",
        "audit_ids": [
            "OfBOuEzrToWzC8aer5miKQ"
        ],
        "methods": [
            "password"
        ],
        "expires_at": "2017-11-01T09:44:05.000000Z",
        "user": {
            "password_expires_at": null,
            "domain": {
                "id": "53bd6f95db004eb3807c66fcbaa45300",
                "name": "magnum"
            },
            "id": "175717d8f4d64b39ba9bc3001e3d8f1c",
            "name": "a6a9cfc8-e8bc-4c6a-a452-4684d98ae925_08b173e0db2049e59d19bdd0e45f6b36"
        }
    }
}
```

### Get Certificates ： 正确的返回

正确的返回是一个json格式的数据，里面有"pem"这一项，刚好是虚拟机初始化时报错的地方

```
-- GET  http://10.10.80.10:9511/v1/certificates/[CLUSTER_ID]

{
    "cluster_uuid": "a6a9cfc8-e8bc-4c6a-a452-4684d98ae925",
    "pem": "-----BEGIN CERTIFICATE-----\nMIIC4jCCAcqgAwIBAgIQR2pl521NS3yFmBrbxWgqdjANBgkqhkiG9w0BAQsFADAZ\nMRcwFQYDVQQDDA5zd2FybS1jbHVzdGVyMjAeFw0xNzEwMzExNjAzMzJaFw0yMjEw\nMzExNjAzMzJaMBkxFzAVBgNVBAMMDnN3YXJtLWNsdXN0ZXIyMIIBIjANBgkqhkiG\n9w0BAQEFAAOCAQ8AMIIBCgKCAQEAuFSJd60hC3UGIFazXBj+GmyeI9gU6a8WJbP8\nUYieqBnmnpt4GlbvTJbzJFQMwL9bqtzbDv0+Hn0O7GIjS6NzncM7fiDkkbCVlXk8\nox7cyJf3KGVuUaHukubJkqX5EMecw2McIPJmfhQQdX6Z5asmHOIU/EfULCoaOtH4\nDptVi7AroQepeB22HIVVR6O4/9vH5r9ET0QPEsu1WnRUZNfniJC7gl9jp1SwDBRD\n+DRtA4g8pg+d8KEw/yueurdV8oN9QupuH/xcwlR2Oi4d6sRio+aefv6/4iATtNo5\nMOLzVK6OtALNiO2j9vRHBZAIjRvkPE8XbFNqeURwu8KX5cogmwIDAQABoyYwJDAS\nBgNVHRMBAf8ECDAGAQH/AgEAMA4GA1UdDwEB/wQEAwICBDANBgkqhkiG9w0BAQsF\nAAOCAQEAmmAgrH3cqCSrCMSqv1HiWBBviguoXx0yLgJoX4Z57DqY6mTf+Zv1dqEf\ngdBe2lpFv2t5CeTk9wjr0iMIbg92KiayUCkty85n/VMa3PWw1iIxc3oCRVX3esNE\ndMvMwaME0iPVnqh8+b6f6dop9OrNDIrGvoovAaUoqgaF+WCYZGBiqMzSw/MIhrjZ\nv/kDf52CjfvFy0N+JSQgVQULr0+U1/Q+KnztrX1g95LMQjcP0mWHag6F4JT2Ty6A\nvZ24sWtj/NXatvZ+aty/eHFt1f4BV9HXho5EPDPkheomZnalrrMKpJk/3jAd+REZ\nbUP+hs4Fa1sUXZuJjsfeXDnQ2hGm+g==\n-----END CERTIFICATE-----\n",
    "bay_uuid": "a6a9cfc8-e8bc-4c6a-a452-4684d98ae925",
    "links": [
        {
            "href": "http://10.10.80.10:9511/v1/certificates/a6a9cfc8-e8bc-4c6a-a452-4684d98ae925",
            "rel": "self"
        },
        {
            "href": "http://10.10.80.10:9511/certificates/a6a9cfc8-e8bc-4c6a-a452-4684d98ae925",
            "rel": "bookmark"
        }
    ]
}
```

### Get Certificates ： 错误的返回

错误返回代码是500，导致我一直以为是magnum代码写的有问题，所以就有了后面一步一步的调试

```
2017-11-02 16:38:33.134 5048 ERROR wsme.api [req-4f45fd22-5b2d-4a24-9e93-08b7bfa95e12 15edc195e4bb4b50bf3c26b0bb5e3b09 - 53bd6f95db004eb3807c66fcbaa45300 - -] Server-side error: "Remote error: BadRequest Invalid input for field 'identity/password/user/password': None is not of type 'string'

Failed validating 'type' in schema['properties']['identity']['properties']['password']['properties']['user']['properties']['password']:
    {'type': 'string'}

On instance['identity']['password']['user']['password']:
    None (HTTP 400) (Request-ID: req-f5ee8f99-7ea3-4679-8f7d-30867ae111a0)
[u'Traceback (most recent call last):\n', u'  File "/usr/lib/python2.7/site-packages/magnum/conductor/handlers/indirection_api.py", line 33, in _object_dispatch\n    return getattr(target, method)(context, *args, **kwargs)\n', u'  File "/usr/lib/python2.7/site-packages/oslo_versionedobjects/base.py", line 184, in wrapper\n    result = fn(cls, context, *args, **kwargs)\n', u'  File "/usr/lib/python2.7/site-packages/magnum/objects/cluster.py", line 139, in get_by_uuid\n    db_cluster = cls.dbapi.get_cluster_by_uuid(context, uuid)\n', u'  File "/usr/lib/python2.7/site-packages/magnum/db/sqlalchemy/api.py", line 212, in get_cluster_by_uuid\n    query = self._add_tenant_filters(context, query)\n', u'  File "/usr/lib/python2.7/site-packages/magnum/db/sqlalchemy/api.py", line 141, in _add_tenant_filters\n    user_name = kst.client.users.get(context.user_id).name\n', u'  File "/usr/lib/python2.7/site-packages/keystoneclient/v3/users.py", line 152, in get\n    user_id=base.getid(user))\n', u'  File "/usr/lib/python2.7/site-packages/keystoneclient/base.py", line 75, in func\n    return f(*args, **new_kwargs)\n', u'  File "/usr/lib/python2.7/site-packages/keystoneclient/base.py", line 349, in get\n    self.key)\n', u'  File "/usr/lib/python2.7/site-packages/keystoneclient/base.py", line 150, in _get\n    resp, body = self.client.get(url, **kwargs)\n', u'  File "/usr/lib/python2.7/site-packages/keystoneauth1/adapter.py", line 288, in get\n    return self.request(url, \'GET\', **kwargs)\n', u'  File "/usr/lib/python2.7/site-packages/keystoneauth1/adapter.py", line 447, in request\n    resp = super(LegacyJsonAdapter, self).request(*args, **kwargs)\n', u'  File "/usr/lib/python2.7/site-packages/keystoneauth1/adapter.py", line 192, in request\n    return self.session.request(url, method, **kwargs)\n', u'  File "/usr/lib/python2.7/site-packages/positional/__init__.py", line 101, in inner\n    return wrapped(*args, **kwargs)\n', u'  File "/usr/lib/python2.7/site-packages/keystoneauth1/session.py", line 578, in request\n    auth_headers = self.get_auth_headers(auth)\n', u'  File "/usr/lib/python2.7/site-packages/keystoneauth1/session.py", line 905, in get_auth_headers\n    return auth.get_headers(self, **kwargs)\n', u'  File "/usr/lib/python2.7/site-packages/keystoneauth1/plugin.py", line 90, in get_headers\n    token = self.get_token(session)\n', u'  File "/usr/lib/python2.7/site-packages/keystoneauth1/identity/base.py", line 89, in get_token\n    return self.get_access(session).auth_token\n', u'  File "/usr/lib/python2.7/site-packages/keystoneauth1/identity/base.py", line 135, in get_access\n    self.auth_ref = self.get_auth_ref(session)\n', u'  File "/usr/lib/python2.7/site-packages/keystoneauth1/identity/v3/base.py", line 167, in get_auth_ref\n    authenticated=False, log=False, **rkwargs)\n', u'  File "/usr/lib/python2.7/site-packages/keystoneauth1/session.py", line 853, in post\n    return self.request(url, \'POST\', **kwargs)\n', u'  File "/usr/lib/python2.7/site-packages/positional/__init__.py", line 101, in inner\n    return wrapped(*args, **kwargs)\n', u'  File "/usr/lib/python2.7/site-packages/keystoneauth1/session.py", line 742, in request\n    raise exceptions.from_response(resp, method, url)\n', u"BadRequest: Invalid input for field 'identity/password/user/password': None is not of type 'string'\n\nFailed validating 'type' in schema['properties']['identity']['properties']['password']['properties']['user']['properties']['password']:\n    {'type': 'string'}\n\nOn instance['identity']['password']['user']['password']:\n    None (HTTP 400) (Request-ID: req-f5ee8f99-7ea3-4679-8f7d-30867ae111a0)\n"].". Detail: 
Traceback (most recent call last):
```


其实在最开始的日志中就提示了问题所在 “ Auth plugin and its options for service user must be provided in [keystone_auth] section. Using values from [keystone_authtoken] section is deprecated.: MissingRequiredOptions: Auth plugin requires parameters which were not given: auth_url”， 不过错误等级是WARNING，所以一直被我忽略掉，后来一步一步调试代码又定位到了这个问题。
解决办法就是配置 /etc/magnum/magnum.conf 中 keystone_auth项 

```
[keystone_auth]
auth_version = v3
auth_uri = http://controller:5000/v3
project_domain_id = default
project_name = service
user_domain_id = default
password = magnum
username = magnum
auth_url = http://controller:35357
auth_type = password
```



