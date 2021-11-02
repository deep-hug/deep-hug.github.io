---
title: 解决git中的冲突
author: DeepHug
index_img: /img/deep-hug-blog-wallpaper.JPG
category: 技术
tags:
  - GIT
excerpt: 使用git仓库，同一分支提交代码时可能会遇到冲突，不同分支合并到同一分支时可能也会遇到冲突，就这两种情况，来解决一下其冲突。
date: 2021-11-02 11:16:24
---

## 第一种情况

### 同一分支，在多人使用的时候，产生冲突

> 每次提交代码前，我们都会`git pull`拉取一下新的代码，当我们看到一下这种情况的时候，本次修改的代码和别人有冲突。

<div>
    <img src="chongtu-1.png" />
</div>

### 那么我们应该怎么做呢？

首先我们正常的去暂存文件和提交更改。

```js
  git add .
  git commit -m '本次的修改'
```

执行完这两行命令之后，要执行`git pull`命令，此时我的`vs code`会变成这样。

<div>
    <img src="chongtu-2.png" />
</div>

此时我们去看下面这张图片上的四个操作

<div>
    <img src="chongtu-3.png" />
</div>

第一个是使用当前修改，第二个是使用传入的修改，第三个是两个都保存，第四个是比较两者的更改。根据需要，保留对应的修改。最后一定要记得`ctrl + s`保存下来。然后再进行暂存和提交更改，最后再推送远程分支。

<div>
    <img src="chongtu-4.png" />
</div>

到此冲突解决完毕。

## 第二种情况

### 多个分支在合同一个分支时产生冲突

分支1(feature/cx1)和分支2(feature/cx2)开发完毕，都要合到master分支。

分支1的操作

```js
  git pull
  git add .
  git commit -m '本次的修改'
  git push
  git checkout master
  git pull
  git merge feature/cx1
  git push
```

> 分支1进行正常的操作即可。

分支2的操作

```js
  git pull
  git add .
  git commit -m '开始'
  git push
  git chekcout master
  git pull
```

**`git merge feature/cx2`在执行这行代码之后，我们的`vscode`会变成这样。**

<div>
    <img src="chongtu-5.png" />
</div>

此时我们去看下面这张图片上的四个操作

<div>
    <img src="chongtu-3.png" />
</div>

第一个是使用当前修改，第二个是使用传入的修改，第三个是两个都保存，第四个是比较两者的更改。根据需要，保留对应的修改。最后一定要记得`ctrl + s`保存下来。然后再进行暂存和提交更改，最后再推送远程分支。

<div>
    <img src="chongtu-4.png" />
</div>

到此冲突解决完毕。