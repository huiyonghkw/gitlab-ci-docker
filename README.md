Gitlab-Docker
==


GitLab is open source software to collaborate on code. 

This recipe based on [sameersbn/docker-gitlab](https://github.com/sameersbn/docker-gitlab) to help you create Gitlab system quickly.


## Feature

- Docker Compose
- PostgreSQL
- Redis
- Gitlab

## Install docker on CentOS 7

Please visit [here](https://github.com/bravist/lnmp-docker)


> 国内用户，推荐使用[阿里云](https://github.com/bravist/gitlab-docker/tree/aliyun)分支安装Docker 


## Usage

1.Clone this repository `gitlab-docker`


```bash
$ git clone https://github.com/bravist/gitlab-docker
```

2.Config the gitlab startup parameters.

```
$ cp .env.example .env
```


3.Go into the `gitlab-docker` directory and build docker images and start up the docker container.

```bash

$ cd gitlab-docker

$ sudo docker-compose build && docker-compose up -d
```

Please allow a couple of minutes for the GitLab application to start. then you can open http://ipaddress:8080. 502. If the server response. Please try to refresh the browser for many times。

## Gitlab backup & restore

Go into the container 

```bash
# This command will create 1494401197_2017_05_10_gitlab_backup.tar on /var/opt/gitlab/backups/
$ sudo gitlab-rake gitlab:backup:create

...
#  Copy the backup file to the server's /mnt/lnmp-docker/gitlab/data/backups
$ sudo cp user@host:/destnation/1494401197_2017_05_10_gitlab_backup.tar user@host:/mnt/lnmp-docker/gitlab/data/backups
...

# This command will overwrite the contents of your GitLab database!
$ sudo gitlab-rake gitlab:backup:restore BACKUP=1494401197_2017_05_10
```

more about: https://github.com/gitlabhq/gitlabhq/blob/master/doc/raketasks/backup_restore.md
