
#### 清除没有执行过 git add 的文件
```
生成一个文件: echo 'test' > 1.log 
清除: git clean -fdx 
Removing 1.log
```
#### 已被git add暂存，但未执行git commit提交，取消暂存
```
查看状态: git status
添加 1.log : git add 1.log
执行命令：git rm --cached -r 1.log
```
#### 已被git add暂存，未执行git commit提交，取消所有文件的暂存
```
git reset
```
#### 未暂存，未执行git commit提交，撤销修改
```
git checkout 1.log
```
#### 已暂存，未执行git commit提交，撤销修改及暂存
```
git reset 1.log 或者 git rm --cached -r 1.log
git checkout 1.log
```
#### 已经 git commit，撤销commit，不撤销git add，不删除改动的代码
```
HEAD~1表示上一次提交 HEAD~2 表示两次提交前
git reset --soft HEAD~1
```
#### 已经 git commit，撤销commit，撤销git add，删除改动的代码
```
HEAD~1表示上一次提交 HEAD~2 表示两次提交前
git reset --hard HEAD~1
```
`HEAD~1` 可以改成提交的`ID(sha1 hash值)`，`ID`也可以简写
```
git log --pretty=oneline
ff4a1ea8ea206bcef41bc2ea8818470497af46c5 (HEAD -> master) 第一次提交
git reset --hard ff4a1ea8ea206bcef41bc2ea8818470497af46c5

git log --pretty=format:"哈希值：%h 作者： %an, 时间：%ar : 备注：%s"
```
