---
layout: post
title:  "Github学习记录"
date:   2018-03-29 12:34:17 +0800
categories: git
---

## 常用命令

- 克隆命令
  - 克隆命令有https和ssh两种方式，例子中为https方式

```
git clone https://github.com/username/program.git
```

- 查看git状态

```
git status
```

- 修改文件后到操作
  - 添加带git仓库
  - 提交文件到git仓库

```
git add filename
git commit -m "annotation"
```

- 推送更新到GitHub上

```
git push origin local:remote
```

- 查看所有分支并判断坐在分支

```
git branch
```

- 创建新分支

注意无论在那个分支上修改代码都要commit，否在在切换时代码会自动合并到被切换的分支
```
git checkout -b new_branch
```

- 合并分支代码到主线

```
git merge --no-ff new_branch
```

