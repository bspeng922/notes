xinetd performs the same function as inetd: it starts programs that provide Internet services. Instead of having such servers started at system initialization time, and be dormant until a connection request arrives, xinetd is he only daemon process started and it listens on all service ports for the services listed in its configuration file. When a request comes in, xinetd starts the appropriate server. Because of the way it operates, xinetd (as well as inetd) is also referred to as a super-server.


+ Install

```shell
$ sudo apt-get install xinetd
```


+ configuration file

/etc/xinetd.conf – The global xinetd configuration file.
/etc/xinetd.d/ directory – The directory containing all service-specific files such as ftp



