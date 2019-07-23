# First Aid Kit Cheat Sheet

Command compilations for different purposes


### Rkt | Containerization



##### List RKT Containers
```
sudo rkt list
```

##### List RKT Images
```
sudo rkt image list
```

##### Fetch RKT Image
```
sudo rkt --insecure-options=image fetch docker://quay.io/zanui/nginx
or
sudo rkt --insecure-options=image fetch filename.aci
```

##### Export RKT Image
```
sudo rkt image export sha512-of-image image-file-name.aci
```

##### Compress Container ACI
```
tar cvf ./filename.aci *
```

##### Extract Container ACI
```
tar xvf ../filename.aci
```

##### Convert manifest file with json formatted text
```
echo "$(cat manifest | jq '.')" > manifest
```

##### Garbage Collect Exited Containers
```
sudo rkt gc --grace-period=0s
```

##### Check Ports Active For Listening
```
netstat -ant | grep -i listen
```

##### View Image Manifest File
```
sudo rkt image cat-manifest sha512-bb82eda7f7aa | less
```

##### Kill Process For Service Restart
```
kill -s HUP 8
```

##### Sample SH File From CouchDB Container
```
#!/bin/bash

sudo systemd-run --slice=machine \
 rkt run \
 --net=host \
 --dns=host \
 --hosts-entry=host \
 --insecure-options=all \
 --set-env=COUCHDB_USER=user \
 --set-env=COUCHDB_PASSWORD=pass \
 --volume data,kind=host,source=/mnt/couchdb_dev/data \
 --mount volume=data,target=/usr/local/var/lib/couchdb \
 registry-1.docker.io/library/couchdb:1.7.1 \
 --name=rs-couchdb-dev
```

### Rkt | Containerization Systemd

##### Systemd Path
```
/etc/systemd/system
```

##### Sample Systemd Service From NodeJs
```
[Unit]
Description=nodejs.service
OnFailure=notify-mq@%i.service

[Service]
LimitNOFILE=65536
Slice=machine.slice
ExecStart=/usr/bin/rkt run \
 --hosts-entry=host \
 --hostname=nodejs01 \
 --net="containernet:IP=1.1.1.1" \
 --net=default \
 --dns=host \
 registry-1.docker.io/library/redis:5.0.3  \
 --volume=volume-data,kind=host,source=/mnt/nodejs_redis_data \
 rs/nodejs:latest  \
 --volume=data,kind=host,source=/mnt/nodejs_data/remotestaff \
 --mount=volume=data,target=/home/remotestaff/remotestaff \
 --working-dir=/home/remotestaff/remotestaff \
 --exec=node -- /home/remotestaff/remotestaff/bin/www \
 --user=remotestaff



ExecStartPre=-/home/core/bin/mq_publish_notif.sh %n starting
ExecStartPost=-/home/core/bin/mq_publish_notif.sh %n started

ExecStopPost=-/home/core/bin/mq_publish_notif.sh %n stopped

ExecStopPost=/usr/bin/rkt gc --grace-period=0s
KillMode=mixed
Restart=always

[Install]
WantedBy=multi-user.target
```

##### Sample Systemd Service From Zend API
```
[Unit]
Description=rs-api.service
OnFailure=notify-mq@%i.service

[Service]
Slice=machine.slice
ExecStart=/usr/bin/rkt run \
        --insecure-options=image \
        --net="containernet:IP=1.1.1.1" \
        --net=default \
        --dns=host \
        --hosts-entry=host \
        --volume=data,kind=host,source=/mnt/api_prod/data/prod_api \
        --mount=volume=data,target=/srv/api \
        rs/api \
        --user=root \
        --interactive=true \
        --name=rs-api

ExecStartPre=-/home/core/bin/mq_publish_notif.sh %n starting
ExecStartPost=-/home/core/bin/mq_publish_notif.sh %n started

ExecStopPost=-/home/core/bin/mq_publish_notif.sh %n stopped
ExecStopPost=/usr/bin/rkt gc --grace-period=0s

KillMode=mixed
Restart=always

[Install]
WantedBy=multi-user.target
```

##### Sample Telegram Notification on /home/core/bin/mq_publish_notif.sh
```
#!/bin/bash

chat_id="-234333232"
hostname="$(hostname)"
text="$2 $1 from $hostname server"
data='{"properties":{},"routing_key":"telegram-message","payload":"{\"chat_id\":\"'$chat_id'\",\"text\":\"'$text'\"}","payload_encoding":"string"}'

/usr/bin/curl -i -u telegram:telegram -H "content-type:application/json" -d "$data" -XPOST http://fl-mq01:15672/api/exchanges/telegram/amq.default/publish
```

## Author

* **Joene Floresca** - [Github](https://github.com/joenefloresca)
