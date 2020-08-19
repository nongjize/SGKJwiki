---
layout: post
author: nong
---
###摘抄
路径：/git使用问题处理汇总
标题：本地仓库无法获取远程仓库分支的处理方法
 
 在拉取远程仓的分支的时候，会报出没有该提交的错误，原因是没有本地没有相应的远程仓分支信息，可以试试使用git fetch之后在拉取远程仓的分支，如果问题还是存在的话就是fetch规则配置的问题，到版本库根目录下面修改配置文件：.git\config  修改内容如下：
 fetch = +refs/heads/*:refs/remotes/origin/*
 
