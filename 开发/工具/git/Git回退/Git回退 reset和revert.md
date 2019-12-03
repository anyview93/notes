# Git回退 reset和revert

今天学习了git回退的两个命令，现在总结一下：

## 1.git reset

![img](.\img\reset.png)	

如果想回退错误的提交C和D，只要把指针移到B上

git reset --hard a0fvf8

而这时候，远程仓库的指针还在D上，如果直接用 git push 将无法推到远程仓库，所以只能用 git push -f 强制推到远程仓库，

这样会有弊端，当你发现C和D不是错误的或者有用的话，将无法找回，因为已经指针远程仓库的指针已经在B上了。

这时候 git revert 就有作用了

## 2.git revert

git revert的作用通过反做创建一个新的版本，这个版本的内容与我们要回退到的目标版本一样，但是HEAD指针是指向这个新生成的版本，而不是目标版本。

![img](.\img\revert.png)

 

单个回退，先revert C，在revert D

git revert 5lk4er
git revert 76sdeb

批量回退

git revert OLDER_COMMIT^..NEWER_COMMIT

 

这样，错误的C和D依然保留，以后可以找得回，最后直接 git push 直接推到远程仓库。

在举个例子：

![img](.\img\revert_batch.png)

错误的B在A和C中间，这时的操作是

先revert B，git revert a0fvf8

在revert C， git revert 76sdeb

再使用 `cherry-pick` 命令将 C 提交重新再生成一个新的提交 C''，这样就实现了将 B提交回退的需求。

## 3.区别：

通过以上对比可以发现，git reset 与 git revert 最大的差别就在于，git reset 会失去后面的提交，而 git revert 是通过反做的方式重新创建一个新的提交，而保留原有的提交。