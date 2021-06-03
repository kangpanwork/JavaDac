[SourceTree](https://www.sourcetreeapp.com/)SourceTree集成了Git Flow功能，能简单方便的操作和实现常规的工作流程。支持OSX和Windows平台。[操作文档](https://support.atlassian.com/sourcetree/)
在 `gitHub` 创建一个仓库 `javaBook`
克隆到本地

```
 > git clone git@github.com:javakangpan/javaBook.git
Cloning into 'javaBook'...
warning: You appear to have cloned an empty repository.
```
添加文件
```
> git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        Bean.md
        CacheLine.md
        Docker.md
        Java8.md

        "\345\234\250\345\256\271\345\231\250\344\270\255\350\277\220\350\241\214Apache\346\234\215\345\212\241\345\231\250.md"
```
添加到暂存区
```
> git add .
warning: LF will be replaced by CRLF in Docker.md.
The file will have its original line endings in your working directory
```
提交
```
> git commit -m "add"
[master (root-commit) d46fbcc] add
 47 files changed, 3182 insertions(+)

```
推送到远程分支
```
> git push
Enumerating objects: 50, done.
Counting objects: 100% (50/50), done.
Delta compression using up to 8 threads
Compressing objects: 100% (50/50), done.
Writing objects: 100% (50/50), 1.92 MiB | 431.00 KiB/s, done.
Total 50 (delta 2), reused 0 (delta 0)
remote: Resolving deltas: 100% (2/2), done.
To github.com:javakangpan/javaBook.git
 * [new branch]      master -> master
```