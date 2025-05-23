1.直接在谷歌搜索ollama，点击第一个链接到官网下载wondows版本的ollama，选择下载路径为D盘。若是直接下载到C盘，我们把它剪切复制到D盘
2.然后在D盘新建一个文件夹ollama
3.在上面的路径输入cmd
4.在命令提示符里面输入
```
ollamasetup.exe /DIR=D:\ollama
```
5.按照屏幕示例点击安装，若是没有，就双击在官网下载的exe。这样我们就把ollama安装到D盘啦

**但是从ollama下载的模型还是默认下载在C盘，你可以看到C盘用户目录下有.ollama文件夹**
##### 修改安装路径
1.在对应的盘符D盘新建文件夹ollamaimagers
2.去环境里面新增环境变量。变量名OLLAMA_MODELS
   变量值为前面新建的文件夹路径D:\ollamaimagers。然后点击确定
3.重启电脑，使环境变量生效

##### 下载模型
去ollama官网下载

![[Pasted image 20250502220949.png]]
找到你想要下载的模型，复制右下角的内容，打开cmd，粘贴回车下载。
要是后面下载很慢，我们可以ctrl c给暂停，然后重新执行。
![[Pasted image 20250502221236.png]]
输入ollama，就可以看到它的相关命令
删除模型 ollama rm 模型名
![[Pasted image 20250502222123.png]]
下载的模型也成功放到了相关路径了
![[Pasted image 20250502222311.png]]