# gitlab-docker
GitLab is open source software to collaborate on code. 

More about [sameersbn/docker-gitlab](https://github.com/sameersbn/docker-gitlab) .


# Feature

- Docker Compose 
- PostgreSQL
- Redis

## Install docker on CentOS 7

Please visit [here](https://github.com/bravist/lnmp-docker)


# Usage

1. clone the `gitlab-docker` on CentOS 7


```bash
$ git clone https://github.com/bravist/gitlab-docker
```


2. go to the gitlab-docker and build it with docker-compose

```bash

$ cd gitlab-docker

$ sudo docker-compose build && docker-compose up -d
```

As default, you can visit http://ipaddress:8080 when the gitlab image built complete. 502. If the server response. Please try to refresh the browser for many times。

### Gitlab backup & restore

visit https://github.com/gitlabhq/gitlabhq/blob/master/doc/raketasks/backup_restore.md, please notice enter the container.

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

