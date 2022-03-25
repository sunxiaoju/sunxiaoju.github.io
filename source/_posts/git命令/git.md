---
layout: post draft
title: git 常用操作命令
date: 2022-03-25 18:58:55
tags: git 
---
### git 常用操作命令

- 创建秘钥：
```js
ssh-keygen -t rsa -C  "xx@163.com"
//然后提示输入密码和确认密码即可完成, 可以直接回车密码默认为空
```
- 查看本机秘钥：
```js
cd ~/.ssh
// 若出现“No such file or directory”,则表示需要创建一个ssh keys。
ls
// 查看文件， 如果有id_rsa、id_rsa.pub就是已经有秘钥存在， 输入pwd可以查看文件所在路径
// 拷贝公钥
cat id_rsa.pub  
```

- 克隆远程仓库代码
```js
git clone git@xxx.git
```

- 查看提交状态
```js
git status
```
- 暂存 提交
```js
git add .
git commit -m ''
```

- 查看git提交记录
```js
git log
```

- 拷贝某次提交到新的分支
```js
git cherry-pick hashxxxx
```


- 查看当前git远程仓库，两种方式：
```js
git remote -v
git remote show origin
```

- 移除与远程仓库的关联：
```js
git remote remove origin
```

- 修改当前仓库地址：
```js
git remote set-url origin git@xxx.git
```

- 本次仓库与远程仓库首次创建关联
```js
git remote add origin git@xx.git
```

- 本地分支重命名
```js
git branch -m oldName  newName
```
- 将重命名后的分支推送到远程
```js
git push origin newName
```
- 删除旧分支
```js
// 删除远程分支
git push --delete origin oldName
// 删除本地分支
git -d oldName
```


- git 回退某次提交
```js
// 会从新生成一个 commit提交    
git revert <commit-id> 
```

###### clone下载分支慢，可以使用这种方式下载

```js
git clone --depth 1 https://github.com/dogescript/xxxxxxx.git
//注意： 此种方式克隆下的代码，切换远程分支的话，需要采用下边方式
git remote set-branches origin 'remote_branch_name'
git fetch --depth 1 origin remote_branch_name
git checkout remote_branch_name
```
