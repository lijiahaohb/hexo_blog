---
title: git基础 
categories: 
- tools
---

## git工作机制
* 工作区---> 代码存放位置
* 缓存区---> 临时存储 `git add`
* 本地库---> 历史版本 `git commit`
* 远程库---> 代码托管中心 `git push`

## git常用命令
### 基本命令
* 设置用户签名
``` bash
git config --global user.name 用户名
git config --global user.email 邮箱 
```

* 初始化本地库
``` bash
git init
```

* 查看本地库状态
``` bash
git status
# 友好的输出格式
git status -sb -uno --show-stash
```

* 添加缓存区
``` bash
git add 文件名
```

* 撤销添加到缓存区的修改
``` bash
git rm --cached 文件名
```

* 提交本地库
``` bash
git commit -m "日志信息"
```

* 提交暂存区(暂存本地不想提交的修改)
``` bash
git stash
```

* 丢弃全部本地修改
``` bash
git checkout -f
```

* 查看版本信息
``` bash
# 查看精简版版本信息
git reflog 
# 查看详细版版本信息
git log --oneline --graph
```

* 版本穿梭
``` bash
git reset --hard 版本号
```

* 标签
``` bash
# 查看标签 
git tag

# 给某个提交贴标签
git tag  标签名
```

### 分支相关命令
* 查看分支
``` bash
# 查看具体的提交信息
git branch -v

# 查看远程分支
git branch -f

# 查看本地和远程的所有分支
git branch -a
```

* 创建分支
``` bash
git branch 分支名
```

* 删除分支
``` bash
git branch -d 分支名
```

* 切换分支
``` bash
# 切换分支
git checkout 分支名

# 创建并且换分支
git checkout -b 分支名
```

* 合并分支
``` bash
git merge 分支名  # 将指定分支与当前分支进行合并
```

* 冲突合并
``` bash
# 当两个版本均有修改时，版本合并时会产生冲突
# 首先需要手动进行合并
# 将修改加入暂存区
git add 文件名
# 提交本地库 注意: 此时提交不能带有文件名，否则会报错
git commit -m "版本信息"
```

### 远程库相关操作
* 为远程库创建别名
``` bash
# 查看已经存在的别名
git remote -v
# 为远程库添加别名
git remote add 别名 远程库地址
# eg: 
git remote add git-demo https://github.com/lijiahaohb/git-demo.git
```

* 更新远程仓库变更
``` bash
git remote update origin
# 之后还要将远程仓库的分支merge到本地仓库
git merge origin/develop 
```

* 将本地库代码推送到远程库
``` bash
git push 别名 本地分支名:远程分支名 
# eg:
git push git-demo master:master
```

* 将远程库代码拉取到本地库
``` bash
git pull 别名 远程分支名:本地分支名
# eg: 
git pull git-demo master:master
```

### git分支管理策略
* master分支: 主干分支，会自动建立
* release分支: 发布分支，realse分支开始测试之后，不能将develop分支内容合并到release分支
* develop分支: 日常开发分支，develop分支代码稳定以后，应该合并回master
* feature分支: 功能分支，为了开发特定功能从develop分支分出，开发完成后需要并入develop分支
* hotfix分支: 软件修复分支，从master分支分出需要合并回master分支，如果存在待发布的release分支，还需要合并到release分支

### git团队开发实践流程
1. git add
2. git commit
3. git pull
4. 解决冲突
5. git push


