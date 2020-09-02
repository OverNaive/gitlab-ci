# GitLab 自动化部署例子

## 步骤一：设置 SSH 无需密码登入服务器

### 在服务器上生成密钥

利用 `ssh-keygen` 命令来生成密钥。
此时在目录 `~/.ssh` 目录下会有如下文件：
- authorized_keys
- id_rsa
- id_rsa.pub
- known_hosts

其中 `id_rsa` 是私钥，`id_rsa.pub` 是公钥。

### 将公钥加入 `authorized_keys` 里

执行命令： `cat ~/.ssh/id_rsa.pub >> authorized_keys`

### 创建 GitLab 变量

1. 进入 GitLab 的 group 里。也可以在项目里配置。
2. 切换 TAB 至 `Settings` => `CI/CD` => `Variables`。
3. 创建变量 `SSH_PRIVATE_KEY`，并将 `id_rsa` 里的内容复制进去。

## 步骤二：拉取代码授权设置

### 设置 Deploy Token

1. 进入 GitLab 的 group 里。也可以在项目里配置。
2. 切换 TAB 至 `Settings` => `Repository` => `Deploy Tokens`。
3. 添加一个 `token` 用于后端部署。
4. 最后得到 `username` 和 `password` (以页面展示为主)

注意：

1. 这里需要设置过期时间，一旦过期了，需要重新生成。
2. 权限请设置为只读（`read_repository`）。

### 如何使用

1. 地址拼接与替换，如：`https://username:password@gitlab.com/group/project.git`
2. 首次克隆，可以直接 `git clone https://username:password@gitlab.com/group/project.git`
3. 已经克隆了，需要修改 remote url：`git remote set-url master https://username:password@gitlab.com/group/project.git`

## 步骤三：项目里的设置

### 设置项目变量

* 进入 GitLab 项目里
* 切换 TAB 至 `Settings` => `Runners` => `Shared runners`，Enable for this project。
* 切换 TAB 至 `Settings` => `CI/CD` => `Variables`。
* 创建变量 `DEPLOY_DIRECTORY`，用于定位项目在服务器上的目录。

### 往项目代码里添加 .gitlab-ci.yml

```yaml
image: ubuntu

stages:
  - deploy

before_script:
  - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
  - eval $(ssh-agent -s)
  - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
  - mkdir -p ~/.ssh
  - chmod 700 ~/.ssh
  - '[[ -f /.dockerenv ]] && echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config'

master-deploy:
  stage: deploy
  script:
    - ssh root@$SERVER_IP "cd $DEPLOY_DIRECTORY && git checkout master && git pull origin master"
  only:
    - master
```

代码来源：[GitLab-examples > ssh-private-key](https://gitlab.com/gitlab-examples/ssh-private-key/-/blob/master/.gitlab-ci.yml)

## 额外：关注自动化部署是否执行成功

进入到项目里的 `CI/CD` 中的 `Pipelines` 里查看执行情况。