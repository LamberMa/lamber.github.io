
## 分离头指针错误

```shell
You are in 'detached HEAD' state. You can look around, make experimental changes and commit them, and you can discard any commits you make in this state without impacting any branched by performing another checkout.
```
说明：你现在的开发内容没有在任何一个分支上，你虽然可以继续开发，但是如果说你哪天要切到别的分支，那们你当前做的操作全部都会丢失。detached head是一种HEAD指针指向了某一个具体的 commit id，而不是分支的情况，此时才会出现这个问题。比如说从远程库clone下来一个远程的repo，clone下来之后，git自动在本地建立了一个本地分支master，并自动与远程库master关联。现在在操作checkout其他分支名(a)。因为本地的工作区目前是刚刚clone的master分支的代码并且与远程关联，但是本机上没有本地分支与远程分支a关联，所以checkout一下就会出现detached head的状态（直接指向了commit id，因为git是离线版本控制，因为此checkout是远程的不是本地的，所以git只能给你一个commit id让你进行操作）。

一般解决方式也比较简单，按照git提示的新建对应的分支即可。

```shell
# 新建分支的同时，checkout过去
git checkout -b <branch_name>
```

