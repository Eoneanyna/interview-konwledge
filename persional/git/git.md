# git命令 
## merge和rebase的区别?

1. rebase把当前的commit放到公共分支的最后面，merge把当前的commit和公共分支合并在一起；
2. 用merge命令解决完冲突后会产生一个commit，而用rebase命令解决完冲突后不会产生额外的commit

## git pull和git fetch的区别？
git pull 是 git fetch + git merge
1. git pull 会拉取所有的提交并合并到当前处理的分支中
2. git fetch会拉取目标分支中所有本地不存在的提交，将这些远程提交存储到本地仓库中，但不会合并在当前本地的分支中