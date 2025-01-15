##### 分支操作
新建分支
```
git branch 分支名
```
注意要是手动创建的仓库，进行初始化后，要进行了提交操作后，才会有那个分支的文档，然后才能进行新建分支的操作

查看当前的分支有哪些
```
git branch -v
```
前面有* 号的表示当前是在这个分支下

切换分支
```
git checkout 要切换到的分支名
```

新建分支并切换到新建的分支
```
git checkeout -b 新建的分支名
```

![[Pasted image 20250114204740.png]]

分支的合并操作，要切换到master
```
git merge 要合并的分支名
```
若是有相同的文件名，就要打开那个文件名，把除了文档内容的其他信息删除掉，保存，然后
```
git add 文件名
git commit -m 信息
```
即可，合并成功之后，切换到“要合并的分支名”那里，打开相同名的文件名可以看到内容是不变的，只有master的文件内容发生变化
![[Pasted image 20250114212612.png]]

##### 标签
标签可以用来创建分支，分支就是引用了一个提交的版本号，标签就是给那个版本增加了一个别名

增加标签
```
git tag 标签名
```
查看标签名
```
git tag
```
![[Pasted image 20250114214256.png]]

##### 远程仓库
如果是从远程仓库里面克隆下来的，那里它的config文档是由url地址的
![[Pasted image 20250114214947.png]]
把url的内容改为gitee克隆里面ssh的内容
![[Pasted image 20250114215656.png]]
本地仓库与远程仓库进行关联
```
git remote add 名称如上面的origin 上面的ssh地址
```
移除操作
```
git remote remove origin
```
重命名操作
```
git remote rename origin
```
也可以直接在config文档里面直接修改

将本地内容推送到远程仓库
```
git push origin(就是上面看到的remote值)
```

如果出现权限问题，在软件上输入
```
ssh-keygen -t rsa -Cgit@gitee.com:xiaoxiaocao8/remote-gitee-test.git
```
-C后面的地址就是ssh的地址
然后去C盘上复制这个ssh的内容
![[Pasted image 20250114221100.png]]
然后去到gitee页面上点击个人页面那个图标，点击设置--SSH公钥，然后在右边的输入框里面粘贴刚刚复制的内容，确定。完成之后再进行git push origin即可
![[Pasted image 20250114221224.png]]

在远程仓库上面修改内容更新到本地
```
git pull origin
```
![[Pasted image 20250114221800.png]]