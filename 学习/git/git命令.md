![[Pasted image 20250114115140.png]]

##### 使用命令行创建仓库
先手动创建一个文件夹如local-test-3,然后右击鼠标打开命令行工具，输入git init就可以对仓库进行初始化了，这中方式里面大部分的文件都是空的，但是用git工具创建的仓库，是默认有一个.gitattributes文件，并提交了的，所有它的文件是有内容的
![[Pasted image 20250114120358.png]]
这里是写有master，但是实际上是没有这个分支的，git的仓库的默认是main
![[Pasted image 20250114120328.png]]

##### 使用命令行克隆远程仓库gitee
在gitee页面选择要克隆的仓库，点击克隆/下载，点击HTTPS，复制链接
![[Pasted image 20250114121222.png]]
在存放仓库的文件夹处，打开命令行，输入
```
git clone 粘贴刚刚复制的链接
```
克隆完成就可以看到一模一样的仓库啦，里面的信息都是一样的
![[Pasted image 20250114121451.png]]
若是想对克隆的仓库名进行更改，直接在命令行后面写上新的仓库名
![[Pasted image 20250114121805.png]]

##### 配置用户名和邮箱
只在这个仓库里面配置
```
git config user.name xiaoxiaocao888
git config user.email 2769260170@qq.com
```
![[Pasted image 20250114123212.png]]
打开config文件，可以看到多了user的内容
![[Pasted image 20250114123555.png]]
所有的仓库都配置
```
git config --global user.name xiaoxiaocao888
git config --global user.email 2769260170@qq.com
```
就可以在本地C盘的用户看到.gitconfig文件里面多了user的内容
![[Pasted image 20250114123122.png]]

##### 在git软件上修改配置信息
法一：点击左上方的file--options
![[Pasted image 20250114123916.png]]

法二：点击左上方的Repository--Repository settings--Git config

![[Pasted image 20250114124049.png]]
查看当前的状态
```
git status
```
工作区增加到暂存区
```
git add 文件名
git commit -m 自己写操作的名称，写上面都可以
```
暂存区返回到工作区
```
git rm --cached 文件名
```
将工作区的所有以.txt结尾的文件增加到暂存区
```
git add *.txt
```
查看历史操作记录
```
git log
```
查看历史操作记录，内容精简版
```
git log --oneline
```

![[Pasted image 20250114195151.png]]
删除文件
![[Pasted image 20250114195404.png]]

##### 误删文件恢复
法一：进行重置，回到删除命令前一步操作，这样做就是日志是看不到那个删除操作的
```
git reset --hard 删除命令前一步操作的版本号
```
![[Pasted image 20250114200820.png]]
法二：还原到提交之前的操作
```
git revert 删除命令的版本号
```
![[Pasted image 20250114202936.png]]
相当于把删除命令前一步的操作重新执行了一次，这个应该是说把那个文件重新下载了


