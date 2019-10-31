# 使用Gitlab Runner 部署Laravel 项目

## 新建部署用户

创建一个deployer用户

```bash
$ sudo useradd deployer
```
修改用户权限，允许其新建目录等命令操作
```bash
$ ls -l /etc/sudoers
-r--r----- 1 root root 3985 10月 30 19:33 /etc/sudoers
```
`etc/sudoers`只有只读的权限，如果想要修改的话，需要先添加w权限

```bash
$ chmod -v u+w /etc/sudoers
```
在最下面增加一个用户
```bash
$vim /etc/sudoers

...
deployer        ALL=(ALL)       NOPASSWD: ALL
```
这时候要记得将写权限收回
```bash
chmod -v u-w /etc/sudoers
```

设置用户密码

```bash
$ passwd deployer
```

登录验证

```bash
$ ssh deployer@your_ip_address
```

生成RSA秘钥对

```bash
$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/deployer/.ssh/id_rsa):
Created directory '/home/deployer/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/deployer/.ssh/id_rsa.
Your public key has been saved in /home/deployer/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:BGGNOhmUvmvaQT0nn3D3iMeISOT036Jj/+/77vqMdYY deployer@iz8vbh3xuahhi5gqllg1u1z
The key's randomart image is:
+---[RSA 2048]----+
|   ...++         |
|    o....        |
|   .o+  .        |
|   +=o .         |
|    +o* S .      |
|   o.. X B o   . |
|    o.. B = . E o|
|   .o.o. o   + o |
|  .o...o...o*B*  |
+----[SHA256]-----+
```

查看私钥

```bash
$ cat /home/deployer/.ssh/id_rsa
```

## 安装 Gitlab Runner

root账户登录

```bash
$ ssh root@your_ip_address
```

添加Gitlab 官方仓库

```bash
# For RHEL/CentOS/Fedora
$ curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh | sudo bash
```

安装Gitlab Runner

```bash
# For RHEL/CentOS/Fedora
$ sudo yum install gitlab-runner
```

查看安装版本

```bash
$ gitlab-runner -v
Version:      12.4.1
Git revision: 05161b14
Git branch:   12-4-stable
GO version:   go1.10.8
Built:        2019-10-28T12:49:57+0000
OS/Arch:      linux/amd64
```

> 更多安装参考官方：https://docs.gitlab.com/runner/install/linux-repository.html

## 新注册Gitlab Runner

登录项目仓库，查看Token并按照下面的配置方式设置参数

![image-20191030161106282](./assets/image-20191030161106282.png)

注册成功后，在服务端和Gitlab中查看结果

```bash
$ gitlab-runner list
```

## 新建`.gitlab.ci.yml`配置文件

切换项目仓库分支为：`develop`，在其中添加`.gitlab-ci.yml`文件，验证内容如下：

```bash
before_script:
  - echo "Before script"
  - cd /var/www/
building:
  stage: build
  script:
    - echo "building..."
testing:
  stage: test
  script:
    - echo "testing..."
deploying:
  stage: deploy
  script:
    - echo "deploying..."
```

提交代码，如果能看到下面内容，简单的CI/CD流程就成功完成了。

![image-20191030172638128](./assets/image-20191030172638128.png)



## 推荐CI/CD 配置

项目、系统、环境等不尽相同，推荐部署Laravel项目按照的`dev`，`test`，`production`三个环境构建项目，仓库分支保持`develop`，`testing`，`master`三个分支对应前面三个环境构建代码，每个环境的应用参数也不同，可以采用`.env.dev`，`.env.test`，`.env.production`保存参数。

```bash
variables:
  RELEASES_STORAGE_DIR: '/var/www/$CI_COMMIT_REF_NAME/$CI_PROJECT_PATH/storage'
  CREATE_RELEASES_STORAGE_DIR: '[ -d $RELEASES_STORAGE_DIR ] || sudo mkdir -p $RELEASES_STORAGE_DIR'
  RELEASES_DIR: '/var/www/$CI_COMMIT_REF_NAME/$CI_PROJECT_PATH/releases'
  CREATE_RELEASE_DIR: '[ -d $RELEASES_DIR ] || sudo mkdir -p $RELEASES_DIR'
  NEW_RELEASES_DIR: '$RELEASES_DIR/$CI_COMMIT_SHORT_SHA'
  CREATE_NEW_RELEASES_DIR: '[ -d $NEW_RELEASES_DIR ] || sudo mkdir -p $NEW_RELEASES_DIR'
  BEFORE_CHMOD: 'sudo chown -R deployer:deployer $NEW_RELEASES_DIR'
  BEFORE_CHMOD_VENDOR: 'sudo chown -R deployer:deployer $NEW_RELEASES_DIR/vendor'
  AFTER_CHMOD: 'sudo chown -R apache:apache /var/www/$CI_COMMIT_REF_NAME && sudo chown -R apache:apache $RELEASES_STORAGE_DIR && sudo chmod -R 777 $RELEASES_STORAGE_DIR'
  CD_NEW_RELEASES_DIR: 'cd $NEW_RELEASES_DIR'
  CD_RELEASES_DIR: 'cd $RELEASES_DIR'
  #Linux删除除了某个文件之外的所有文件/目录
  CLEAN_RELEASES_DIR: 'ls |grep -v $CI_COMMIT_SHORT_SHA |xargs sudo rm -rf'
  RM_RELEASE_STORAGE_DIR: 'sudo rm -rf $NEW_RELEASES_DIR/storage'
  LN_RELEASE_STORAGE_DIR: 'sudo ln -nfs $RELEASES_STORAGE_DIR $NEW_RELEASES_DIR/storage'
  LN_RELEASE_DIR: 'sudo ln -nfs $NEW_RELEASES_DIR /var/www/$CI_COMMIT_REF_NAME/$CI_PROJECT_PATH/current'
  MV_REPO: 'sudo mv -fv /home/deployer/$CI_PROJECT_DIR/* $NEW_RELEASES_DIR'
  CP_DEV_ENV: 'cp /home/deployer/$CI_PROJECT_DIR/.env.dev $NEW_RELEASES_DIR/.env'
  CREATE_FRAMEWORK_CACHE: '[ -d $RELEASES_STORAGE_DIR/framework/cache ] || sudo mkdir -p $RELEASES_STORAGE_DIR/framework/cache'
  CREATE_FRAMEWORK_SESSIONS: '[ -d $RELEASES_STORAGE_DIR/framework/sessions ] || sudo mkdir -p $RELEASES_STORAGE_DIR/framework/sessions'
  CREATE_FRAMEWORK_TESTING: '[ -d $RELEASES_STORAGE_DIR/framework/testing ] || sudo mkdir -p $RELEASES_STORAGE_DIR/framework/testing'
  CREATE_FRAMEWORK_VIEWS: '[ -d $RELEASES_STORAGE_DIR/framework/views ] || sudo mkdir -p $RELEASES_STORAGE_DIR/framework/views'


before_script:
  - echo "Before script"
  - echo $CI_COMMIT_REF_NAME
  - echo $CI_PROJECT_PATH
  - echo $CI_COMMIT_SHORT_SHA
  - echo $CI_REPOSITORY_URL
  - echo $CI_PROJECT_DIR
  - 'eval $CREATE_RELEASES_STORAGE_DIR'  # will execute
  - 'eval $CREATE_RELEASE_DIR'  # will execute
  - 'eval $CREATE_NEW_RELEASES_DIR'  # will execute
  - 'eval $CD_NEW_RELEASES_DIR'


stages:
  - build
  - test
  - deploy-dev

building:
  stage: build
  script:
    - echo "Move repo..."
    - echo $NEW_RELEASES_DIR
    - 'eval $BEFORE_CHMOD'
    - 'eval $MV_REPO'
    - composer install
    - 'eval $BEFORE_CHMOD_VENDOR'

testing:
  stage: test
  script:
    - echo "testing..."
    # - php ./vendor/bin/phpunit

deploying_dev:
  stage: deploy-dev
  script:
    - echo "deploying dev..."
    - 'eval $CP_DEV_ENV'
    - php artisan key:generate
    - 'eval $CREATE_FRAMEWORK_CACHE'
    - 'eval $CREATE_FRAMEWORK_SESSIONS'
    - 'eval $CREATE_FRAMEWORK_TESTING'
    - 'eval $CREATE_FRAMEWORK_VIEWS'
    - php artisan cache:clear
    - php artisan config:clear
    - php artisan storage:link
    - php artisan migrate --force
    - php artisan passport:keys
    - echo "Restarting supervisor"
    - sudo supervisorctl restart all
    - echo "Linking storage directory"
    - 'eval $RM_RELEASE_STORAGE_DIR'
    - 'eval $LN_RELEASE_STORAGE_DIR'
    - echo 'Linking current directory'
    - 'eval $AFTER_CHMOD'
    - 'eval $LN_RELEASE_DIR'
    - echo 'Removing earlier app'
    - 'eval $CD_RELEASES_DIR'
    - 'eval $CLEAN_RELEASES_DIR'
  only:
    - develop

```

参考

> 1. [Test and deploy Laravel applications with GitLab CI/CD and Envoy](https://docs.gitlab.com/ce/ci/examples/laravel_with_gitlab_and_envoy/index.html)
> 2. [GitLab CI/CD environment variables](https://docs.gitlab.com/ee/ci/variables/README.html)
> 3. https://github.com/bravist/gitlab-ci-docker
> 4. https://github.com/tuandm/laravue/blob/master/Envoy.blade.php



## 可能的问题

### 1.  is this a git repository?

> Gitlab Runner Nginx 自定义端口后出现无法克隆仓库，Gitlab issue: [unable to access with 8080 web port](https://gitlab.com/gitlab-org/gitlab-runner/issues/1148)

```bash
Running with gitlab-runner 12.4.1 (05161b14)
  on Gitlab Runner For Deploy 4LgtuQbB
Using SSH executor...
Running on iz8vbh3xuahhi5gqllg1u1z via iz8vbh3xuahhi5gqllg1u1z...
Fetching changes with git depth set to 50...
warning: templates not found builds/4LgtuQbB/0/zhangqiuzhe/api.tmp/git-template
Reinitialized existing Git repository in /home/deployer/builds/4LgtuQbB/0/zhangqiuzhe/api/.git/
fatal: http://gitlab-ci-token:[MASKED]@gitlab.higgses.com/zhangqiuzhe/api.git/info/refs not valid: is this a git repository?
ERROR: Job failed: Process exited with: 1. Reason was:  ()

```

修改Gitlab 内置Nginx监听端口号，比如设置：`8888`

```bash
vi  /var/opt/gitlab/nginx/conf/gitlab-http.conf

##
###################################
##         configuration         ##
###################################


server {
  listen *:8888;


  server_name your_domain;
...
```

修改`external_url`端口号，比如设置：`8888`

```bash
vim /etc/gitlab/gitlab.rb

## GitLab configuration settings
##! This file is generated during initial installation and **is not** modified
##! during upgrades.
##! Check out the latest version of this file to know about the different
##! settings that can be configured by this file, which may be found at:
##! https://gitlab.com/gitlab-org/omnibus-gitlab/raw/master/files/gitlab-config-template/gitlab.rb.template


## GitLab URL
##! URL on which GitLab will be reachable.
##! For more details on configuring external_url see:
##! https://docs.gitlab.com/omnibus/settings/configuration.html#configuring-the-external-url-for-gitlab
external_url 'http://your_domain:8888'
...
```



修改了配置需要更新并重启

```bash
$ gitlab-ctl reconfigure

$ gitlab-ctl restart
```



###  2. Pipeline 不执行

Push 代码后Pipelines 卡住不执行，提示没有可以使用的 runner 。如果出现这种情况，需要在注册成功的 runner 中编辑一下设置，运行未标记的作业需要勾选

![image-20191030165624205](./assets/image-20191030165624205.png)

![image-20191030165643847](./assets/image-20191030165643847.png)
