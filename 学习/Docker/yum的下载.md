执行以下步骤
```
wget http://mirrors.aliyun.com/centos/7/os/x86_64/Packages/yum-cron-3.4.3-168.el7.centos.noarch.rpm
wget http://mirrors.aliyun.com/centos/7/os/x86_64/Packages/yum-metadata-parser-1.1.4-10.el7.x86_64.rpm
wget http://mirrors.aliyun.com/centos/7/os/x86_64/Packages/python-urlgrabber-3.10-10.el7.noarch.rpm
wget http://mirrors.aliyun.com/centos/7/os/x86_64/Packages/python-2.7.5-89.el7.x86_64.rpm
wget http://mirrors.aliyun.com/centos/7/os/x86_64/Packages/rpm-libs-4.11.3-45.el7.x86_64.rpm
wget http://mirrors.aliyun.com/centos/7/os/x86_64/Packages/rpm-python-4.11.3-45.el7.x86_64.rpm
wget http://mirrors.aliyun.com/centos/7/os/x86_64/Packages/libxml2-2.9.1-6.el7.5.x86_64.rpm
wget http://mirrors.aliyun.com/centos/7/os/x86_64/Packages/python-libs-2.7.5-89.el7.x86_64.rpm
wget http://mirrors.aliyun.com/centos/7/os/x86_64/Packages/rpm-4.11.3-45.el7.x86_64.rpm
wget http://mirrors.aliyun.com/centos/7/os/x86_64/Packages/yum-3.4.3-168.el7.centos.noarch.rpm
wget http://mirrors.aliyun.com/centos/7/os/x86_64/Packages/yum-plugin-fastestmirror-1.1.31-54.el7_8.noarch.rpm

rpm -ivh python-libs-2.7.5-89.el7.x86_64.rpm
rpm -ivh python-2.7.5-89.el7.x86_64.rpm

rpm -ivh libxml2-2.9.1-6.el7.5.x86_64.rpm
rpm -ivh python-urlgrabber-3.10-10.el7.noarch.rpm
rpm -ivh yum-metadata-parser-1.1.4-10.el7.x86_64.rpm

rpm -Uvh yum-3.4.3-168.el7.centos.noarch.rpm yum-plugin-fastestmirror-1.1.31-54.el7_8.noarch.rpm

yum --version
```
有时候可能还要yum udate