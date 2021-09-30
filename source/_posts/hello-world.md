---
title: Hexo + Github + Travis CI 搭建博客
---
使用Hexo搭建个人博客其实已经有一段时间，但是由于种种原因被搁置了很久（~PS：其实就是因为太懒了）。
明天就是国庆节了，赶在假期前给自己加加班，哈哈哈！
好了，闲话不多说，我们直接上干货！

## 环境准备
 - [nodejs](https://nodejs.org/en/)  在官网找到对应系统版本下载即可，前端的小伙伴应该都不陌生了~
 - [git](https://git-scm.com/)  同上，下载安装！！！程序员必备技能，如果没有学过的小伙伴，建议先学一学哦！
 - [github](https://github.com/)  开源万岁！

## Github
在github上建立一个`username.github.io`的仓库（~ps：这里username并非真的是username，而是你的github用户名，例如我的就是：xiaoming985.github.io，切记要用用户名同名哦！）
## Hexo
下载安装好nodejs之后，打开cmd命令行，使用如下命令安装hexo-cli
```bash
$ npm install -g hexo-cli -g
```
安装好之后，使用hexo -v查看是否成功
```bash
$ hexo -v
```
然后，开始创建本地博客环境（folderName指定项目名称，可选，不指定，则在当前路径下搭建初始化项目）
```bash
$ hexo init [folderName]
```
执行成功后，启动项目（简写hexo s），然后你就可以在浏览器中输入[localhost:4000](http://localhost:4000)看到你的博客啦！
```bash
$ hexo server
```
生成静态页面文件（简写hexo g），使用该命令可以将编写好的博文转成静态页面
```bash
$ hexo generate
```
然后修改项目配置文件`_config.yml`，找到deploy修改如下（repo url要用你上面建立的仓库的地址哦!）
```yml
deploy:
  type: git
  repo:  https://...... # repo url
  branch: main
```
然后部署(简写hexo d)，运行成功后，你就可以通过浏览器打开 https://username.github.io 看到你的博客了~
```bash
hexo deploy
```

## Travis CI自动部署
细心的小伙伴可能已经发现了，github远程仓库上并没有保存我们编写的博客源码，而是保存了构建后的静态页面。
那么，问题来了，如果我们换了个环境呢？（比如换了一部电脑，我想继续写博客怎么办？）
我能不能既保存源码，又部署静态页面呢？（答案是可以的，小孩子才做选择！）
- 用github托管源码，然后我们就可以在多个环境中通过clone获取我们的博客源码了，妙啊！
- 然后通过新分支来部署我们的博客页面，真实妙蛙种子到了米奇妙妙屋，秒到家了！

步骤：
 - 将博客源码使用git上传到远程仓库，这里你可以上传至一个新的独立仓库，也可以直接使用`username.github.io`进行托管
 - 然后将 [Travis CI](https://github.com/marketplace/travis-ci) 添加到你的 GitHub 账户中。
 - 前往 GitHub 的 [Applications settings](https://github.com/settings/installations)，配置 Travis CI 权限，使其能够访问你的 repository。
 - 接下来，在 github -> setting -> Developer Settings -> Personal access tokens 中点击Generate new token按钮生成一个token（只勾选 repo 的权限即可），生成后记得要copy下来哦，等下会用到。切记，不要泄漏！[Personal access tokens传送门](https://github.com/settings/tokens)
 - 回到 Travis CI，前往你的 repository 的设置页面，在 Environment Variables 下新建一个环境变量，Name 为 `GH_TOKEN`，Value 为刚才你在 GitHub 生成的 Token。确保 DISPLAY VALUE IN BUILD LOG 保持 不被勾选 避免你的 Token 泄漏。点击 Add 保存。
 - 回到你的博客项目，在项目根目录下添加`.travis.yml`配置文件，配置如下：
```yml
# 指定语言环境
language: node_js
# 指定需要sudo权限
sudo: false # required
# 指定node_js版本
node_js: 
  - 12
# 指定缓存模块，可选。缓存可加快编译速度。
cache: npm

# 指定博客源码分支，因人而异。hexo博客源码托管在独立repo则不用设置此项
branches:
  only:
    - main 

before_install:
  - npm install -g hexo-cli

# Start: Build Lifecycle
install:
  - npm install
  - npm install hexo-deployer-git --save

# 执行清缓存，生成网页操作
script:
  - hexo clean
  - hexo generate

deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GH_TOKEN
  keep-history: true
  on:
    branch: main
  local-dir: public
# End: Build LifeCycle
```
 - 然后修改`_config.yml`，如下：(替换username即可)，以下配置即构建后将静态文件推送指`gh-pages分支`
```yml
deploy:
  type: git
  repo: https://gh_token@github.com/username/username.github.io.git
  branch: gh-pages
```
 - 完成后，将修改后的`_config.yml`以及`.travis.yml`推送指远程托管仓库，随后，Travis CI就会自己开始工作了，稍等片刻便可以发现远程仓库多了一个新的分支`gh-pages`
 - 在 GitHub 中前往你的 repository 的设置页面，修改 GitHub Pages 的部署分支为 gh-pages。
 - 稍等片刻，你就又能通过浏览器打开 https://username.github.io 看到你的博客啦~

## 更多的项目展示
有的小伙伴可能又有问题了，github.io用来部署博客了，那我还能部署别的项目吗？答案是可以的，将我们构建（`npm run build`等）好之后的项目，推送到远程的`gh-pages`分支即可。开搞！
- 使用自动部署，可参考如下配置：
```yml
# 指定语言环境
language: node_js
# 指定需要sudo权限
sudo: false # required
# 指定node_js版本
node_js: 
  - 12
# 指定缓存模块，可选。缓存可加快编译速度。
cache: 
  directories:
    - node_modules

# 指定博客源码分支，因人而异。hexo博客源码托管在独立repo则不用设置此项
branches:
  only:
    - main 

before_install:
  - npm install -g yarn

# Start: Build Lifecycle

install:
  - yarn

# 执行清缓存，生成网页操作
script:
  - yarn build

# 设置git提交名，邮箱；替换真实token到_config.yml文件，最后deploy部署
after_script:
  

# End: Build LifeCycle

deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GH_TOKEN
  local-dir: ./dist
  keep-history: true
  on:
    branch: main
  local-dir: public
```
- 手动推送，如下命令，即将构建好的`dist`目录推送到远端的`gh-pages`分支
```bash
$ git subtree push --prefix=dist origin gh-pages
```
- PS：手动部署有小注意点
  - `/dist` 目录需要被 git 记录，于是后面我们才可以用它作为子树（subtree），因此 `/dist` 不能被 `.gitignore` 规则排除
  - dist 代表子树所在的目录名
  - origin 是 remote name
  - gh-pages 是目标分支名称


