### git拉取远程代码，但不要覆盖本地新修改的代码
参考连接
https://blog.csdn.net/qq_42956653/article/details/121613703
https://blog.csdn.net/Joye_7y/article/details/125769826
1.先切换到自己的分支
```bash
git checkout 自己的分支名
```
2.在镇江分支上，先提交
```bash
git add .
```
3.然后，将提交的代码放到暂存区
```bash
git stash
```
4.切换到主分支  
```bash
git checkout master
```
5.拉取远程master代码
```bash
git pull orign master
```
6.创建一个新分支
```bash
git checkout -b 新分支名
```
7.现在，将刚刚暂存区的代码放回本地
```bash
git stash pop
```
8.最后，正常提交代码

