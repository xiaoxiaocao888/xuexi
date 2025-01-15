在IDEA处新建一个空白项目，生成后新建a.txt文件，点击上方的VCS，点击Share Project on GitHub
![[Pasted image 20250113222806.png]]
写入要在GitHub上创建的仓库名，点击Add account,点击Log in via GitHub,然后会跳转到一个页面，点授权，写入对应的密码后返回软件，点击share
![[Pasted image 20250113222923.png]]
然后只勾选a.txt文件，点击Add
![[Pasted image 20250113223348.png]]
打开github的网站即可看到对应的仓库名，点击仓库名就可以看到a.txt文件
![[Pasted image 20250113223533.png]]

在idea处对文件进行修改，上传到本地仓库：在对应的文件右击鼠标，点击Git，点击Commit File
![[Pasted image 20250113223746.png]]
在框里面写入提示信息，点击Commit，若是点击Commit and Push就是在本地和远程仓库同时更新
![[Pasted image 20250113224005.png]]
这里对a.txt文件进行修改为cccc，然后选择Commit and Push,点击Push，在GitHub页面更新可以看到a.txt文件内容变为cccc
![[Pasted image 20250113224523.png]] 在远程仓库Github对a.txt文件进行修改并提交，在idea处点击Git，点击Pull，点击Pull，等一会再打开，可以看到a.txt文件的内容和远程仓库的一样了
![[Pasted image 20250113224948.png]]

其他人要下载远程仓库的内容，就是idea处点击上方的Git，点击Clone，然后点击github,填入仓库的链接克隆即可
![[Pasted image 20250113225233.png]]

#### 与gitee
点击File，点击Settings，点击Plugins,在搜索框中输入gitee，在对应的软件点击install，下载插件
![[Pasted image 20250113225626.png]]
其他的操作和github的操作是一样的，点击Git，点击Gitee，点击share project on gitee
![[Pasted image 20250113230505.png]]