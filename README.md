Gitlab-Docker for Aliyun
==


GitLab is open source software to collaborate on code. 

This recipe based on [sameersbn/docker-gitlab](https://github.com/sameersbn/docker-gitlab) to help you create Gitlab system quickly.


## Feature

- Docker Compose
- PostgreSQL
- Redis
- Gitlab

> Docker要求64位的系统且内核版本至少为3.10

## Install docker on Aliyun ECS

1.添加yum源

```
# yum install epel-release –y

# yum clean all
```

2.安装并运行Docker

```
# yum install docker-io –y

# systemctl start docker
```
3.检查安装结果

```bash
# docker info
```

4.针对Docker客户端版本大于1.10的配置阿里云加速器

```bash
$ sudo mkdir -p /etc/docker
$ sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://muehonsf.mirror.aliyuncs.com"]
}
EOF

$ sudo systemctl daemon-reload
$ sudo systemctl restart docker

```


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


##References

+ [ECS上搭建Docker(CentOS7)
](https://help.aliyun.com/document_detail/51853.html?spm=5176.8208715.110.1.ps6bxJ)
+ [使用加速器将会提升您在国内获取Docker官方镜像的速度！
](https://cr.console.aliyun.com/?spm=5176.2020520152.210.d103.H4Rlih#/accelerator)
