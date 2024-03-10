---
title: 配置多个 git 账户
description: "同一个电脑配置多个 github 账户、gitLab 账户，或者二者混合，备忘"
date: 2024-03-10T16:06:00
thumb: "git-multiple-account.png"
tags:
    - git
    - multiple account
    - tips
---

同一个电脑配置多个 github 账户、gitLab 账户，或者二者混合，备忘。

## 1. 同一个电脑怎么配置多个github账户？
如何配置[参考](https://medium.com/@trionkidnapper/ssh-keys-with-multiple-github-accounts-c67db56f191e)
或者墙内[点这里](https://www.cnblogs.com/SZLLQ2000/p/10037204.html)
按以上配置以后，在windows下还要配置 [auto-launching](https://help.github.com/articles/working-with-ssh-key-passphrases/)。

完成后，就可以用不同账户的权限去提交代码了。但是提交时候的 `user.name`、 `user.email` 用的是全局的。即使你用不同人的账户权限提交了代码，但提交的记录中的用户是你全局配置的那个。

那么不同仓库如何用不同的用户名提交？

只需要在这个仓库下单独设置：
```sh
git config --local user.name 'xxx'
git config --local user.email 'xxx@xx.com'
```
更多信息参考 [git config](https://git-scm.com/docs/git-config)。


## 2. 同一个电脑如何配置访问不同 git 服务器？如 github 和 gitLab
这个时候可以共用一个 key。但是如果要区分账户提交的话，还要参考1中后半部分修改。
