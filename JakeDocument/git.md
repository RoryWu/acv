# Git

git add . 

git status

git commit -a 

git fetch origin

git rebase origin 

-------------------

没有问题:

>  git push

--------------

有问题: 

> 处理冲突
>
> git add
>
> git rebase --continue
>
> git push



如果中间有提交重复

> git rebase --abort 
>
> git status
>
> 现在已经取消了上一次的rebase
>
> 修改冲突
>
> git add
>
> git rebase --continue
>
> git push 

