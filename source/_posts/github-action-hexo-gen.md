---
title: 在GitHub Action里设置 Hexo gen 的折腾笔记
date: 2019-10-17 22:44:00
updated: 2019-10-17 22:44:00
tags: [github,action,nodejs,npm,hexo]
---

不想阅读Github Action厚重的文档，用预配置好的NPM Action折腾了半天，终以失败告终。由于不熟悉NPM和Node.js，最后爆出的错误摸不到头脑，就此作罢。

寻找另一个方法，在 `https://github.com/user/repo/actions/new` 中选择了跳过，自己设置一个空白的Workflow。

1. 为了避免Action重新推送仓库后循环触发Action（不知Github有没有对这种情况进行特殊处理）添加路径过滤器，仅当 `/source` 目录存在更新时才触发 Workflow

2. 在仓库的 `Settings->Deploy keys` 里设置Github Action push时使用的SSH公钥，并赋予写权限，`Settings->Secrets` 里设置私钥的 Base64 编码。
私钥会通过环境变量传入Action的shell，`base64 -d` 即可解码，base64编码是为了避免潜在的回车换行符问题（环境变量里出现的换行符似乎不能正确地写入文件）

3. 注意工作目录的问题，别瞎跑，目录会丢掉的

4. `~/.ssh/id_rsa` 文件注意设定权限 `0600` 默认的权限过于宽松，SSH会不读取。

5. `~/.ssh/config` 里需写入 
```
Host *
    StrictHostKeyChecking no
```
否则可能会弹出要求确认SSH fingerprint的交互消息。

6. 记得设定 `git config --global user.name/email` 否则无法提交
7. 如果使用了自定义的域名，注意 `CNAME` 文件是否在 `hexo clean` 后被删除，可能需要自己写回去。
9. 注意部分Hexo主题可能以 git submodule 方式引入， Github Action 克隆仓库时不会自动克隆子模块，导致生成的所有页面空白，Hexo只会给出警告而不是错误。


### 附上自己写的workflow（糊成一坨，凑合能用）

```yaml
name: Generate Hexo

on:
  push: 
    paths: 
      - 'source/**'

jobs: 
  build: 
    name: Refresh
    runs-on: ubuntu-latest
    steps:
      - name: Install Node.js and NPM
        run: |
          sudo apt-get install curl -y
          curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
          sudo apt-get install nodejs -y
          echo '=====Show NPM version====='
          npm -v
          echo '=====Show Node.js version====='
          node -v
      - name: Install Hexo
        run: |
          sudo npm install -g hexo-cli
          echo '=====Show HEXO version====='
          hexo -v
      - name: Prepare Key and SSH config
        run: |
          mkdir ~/.ssh
          echo $DEPLOY_PRIVKEY | base64 -d > ~/.ssh/id_rsa
          chmod 0600 ~/.ssh/id_rsa
          echo 'Host *' >> ~/.ssh/config
          echo '    StrictHostKeyChecking no' >> ~/.ssh/config
          chmod 0600 ~/.ssh/config
        env: 
          DEPLOY_PRIVKEY: ${{ secrets.DEPLOY_PRIVKEY }}
      - name: Clone git repo
        run: |
          cd ~
          echo '=====Show work path====='
          pwd
          git clone git@github.com:cpdyj/blog.git
          cd blog
          ls
          echo '=====Show work path====='
          pwd
          cat ./package.json
          npm install
          npm audit fix
          hexo clean
          hexo g
          echo "blog.iseki.space" > ./docs/CNAME
          git config --global user.name "User name"
          git config --global user.email "Email@example.com"
          git add docs
          git commit -am "Auto generate on Github Action at `date`"
          git push
```
