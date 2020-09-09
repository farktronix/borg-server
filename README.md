# Borg Backup Server Container
![alt text](https://borgbackup.readthedocs.io/en/stable/_static/logo.png "Borgbackup")

### Description

My take on a Borgbackup Server as a Docker container to faciliate the backing up of remote machines using [Borgbackup](https://github.com/borgbackup)

### Dockerfile
```
FROM alpine:latest
MAINTAINER b3vis
#Install Borg & SSH
RUN apk add openssh sshfs borgbackup supervisor --no-cache --repository http://dl-3.alpinelinux.org/alpine/edge/testing/
RUN adduser -D -u 1000 borg && \
    ssh-keygen -A && \
    mkdir /backups && \
    chown borg.borg /backups && \
    sed -i \
        -e 's/^#PasswordAuthentication yes$/PasswordAuthentication no/g' \
        -e 's/^PermitRootLogin without-password$/PermitRootLogin no/g' \
        -e 's/AuthorizedKeysFile.*/AuthorizedKeysFile  \.ssh\/authorized_keys \/etc\/authorized_keys\/%u/' \
        /etc/ssh/sshd_config
COPY supervisord.conf /etc/supervisord.conf
RUN passwd -u borg
EXPOSE 22
CMD ["/usr/bin/supervisord"]
```


### Usage

* Container Creation:
```
docker create \
  --name=borg-server \
  --restart=always \
  -v path/to/authorized_keys_file:/etc/authorized_keys/borg \
  -v path/to/backups:/backups \
  -p 2022:22 \
  b3vis/borg-server
```
