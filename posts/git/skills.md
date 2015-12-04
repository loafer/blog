##git stash & git stash pop
不知是否遇到过这样的事，当你正在修改一个bug时，突然有个更紧急的bug需要你去修复，如果保存之前的修改？
+ 1、在a分支上的修改你可以先git stash，隐藏你在a分支上的修改
+ 2、然后基于服务器创建b分支 例如git checkout -b b origin/branch，这时候到b分支上bug fixed并提交；这部很重要，要基于服务器来创建，如果你基于a分支来创建，那么会包含a分支的修改。这显然不是我们希望的结果。
+ 3、然后git checkout a回到a分支
+ 4、然后git stash pop，继续刚刚a分支的修改
