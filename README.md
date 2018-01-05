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

## [Gitlab backup & restore](https://github.com/sameersbn/docker-gitlab#creating-backups)

```bash
$ docker exec -it gitlabdocker_gitlab_1 sh

# backup, for example file: 1515124070_2018_01_05_9.2.2_gitlab_backup.tar
$ cd /home/git/gitlab && bundle exec rake gitlab:backup:create SKIP= RAILS_ENV=production

# restore
$ cd /home/git/gitlab && bundle exec rake gitlab:backup:restore
```
