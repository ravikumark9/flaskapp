# flaskapp


To launch docker containers in remote machine:

https://docs.docker.com/install/linux/linux-postinstall/#configure-where-the-docker-daemon-listens-for-connections

Under /etc/systemd/system/multi-user.target.wants/docker.service
Replace
#ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
ExecStart=/usr/bin/dockerd -H tcp://172.31.22.94:2375 -H unix:///var/run/docker.sock
Systemctl daemon-reload
Systemctl restart docker
[root@server ~]# netstat -lntp | grep dockerd
tcp        0      0 172.31.22.94:2375       0.0.0.0:*               LISTEN      11641/dockerd       
[root@server ~]# 

In aws security groups open the port 2375

2375 is unsecured, 2376 is secured

To update the new taskdefinition and service which is having new image:
In update-service, --force-new-deployment won't work effectively. So we have to set Minimum health percent - 0.





