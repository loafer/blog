### git remote
为了便于管理，Git要求每个远程主机都必须指定一个主机名。git remote命令就用于管理主机名。

不带选项时后，git remote　列出所有远程主机
```shell
$ git remote
origin
```

使用-v选项，列出主机名和对应的网址
```shell
$ git remote -v
origin https://github.com/quick4j/quick4j-webjs.git (fetch)
origin https://github.com/quick4j/quick4j-webjs.git (push)
```
上面命令结果显示当前只有一台远程主机，叫做 origin，以及它的网址。
克隆版本库时，Git会自动将远程主机命名为 origin。如果需要更改主机名请使用'-o' 选项指定
```shello
$ git clone -o quick4j https://github.com/quick4j/quick4j-framework.git
$ git remote
quick4j
```

git remote add 用于添加远程主机
```shell
git remote add origin git@github.com:quick4j/quick4j-framework.git
```

git remote rm 用于删除远程主机
```shell
$ git remote rm origin
```

git remote rename 用于更改远程主机名
```shell
$ git remote rename origin quick4j
```
以上命令将远程主机名由 origin 变更为 quick4j。

###git branch

不带选项的时候，git branch　列出所有分支。
```shell
$ git branch
* develop
  master
```
上面命令结果中带'*'表示当前所处的分支。

使用-a选项，可以查看所有分支（包括远程分支，红色表示）
```shell
$ git branch -a
* develop
  master
  remotes/origin/HEAD -> origin/master
  remotes/origin/develop
  remotes/origin/master
```

使用-d选项，删除完全合并的本地分支
```shell
$ git branch -d fixbug
```

使用-D选项，删除未合并的本地分支
```shell
$ git branch -D fixbug
```
使用-m选项，可以重新命名分支
```shell
$ git branch -m fixbug hotfix
```
以上命令将分支 fixbug 重命名为 hotfix。

如若**删除远程分支**，需使用　git push origin 命令，并带上--delete选项
```shell
$ git push origin --delete hotfix
```
以上命令为删除当前版本库的 hotfix 远程分支。
