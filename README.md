# Django-xadmin-centos7项目部署

##### 大家好，抽点时间跟大家说说我在Django+xadmin项目服务器部署遇到的一些问题以及自己的一点收获，希望初次部署Django的你可以少走点弯路，话不多说，直接切入主题（以下所有的配置都是在root用户下完成）
#### 第一步：更新系统软件包（二选一）
```
yum -y update	(所有都升级和改变)
升级所有包,系统版本和内核，改变软件设置和系统设置

yum -y upgrade	(不变内核和设置,升级包和系统版本)
升级所有包和系统版本，不改变内核,软件和系统设置
```
#### 第二步：安装软件管理包和可能使用的依赖
```
 1. yum -y groupinstall "Development tools"
 2. yum install openssl-devel bzip2-devel expat-devel gdbm-devel readline-devel sqlite-devel
```
###### 如果安装失败执行如下命令
```
 3. yum clean all 
 4. yum -y groupinstall "Development tools"
 5. yum install openssl-devel bzip2-devel expat-devel gdbm-devel readline-devel sqlite-devel
```
#### 第三步：下载Pyhton3到 /usr/local 目录 
###### 1. 切换到 /usr/local 目录，获取压缩包，解压，进入python-3.6.6文件夹
```
 6. cd /usr/local
 7. wget https://www.python.org/ftp/python/3.6.6/Python-3.6.6.tgz
 8. tar -zxvf Python-3.6.6.tgz
 9. cd Python-3.6.6
```
###### 2. 编译安装python3到指定路径，安装python3
```
 10. ./configure --prefix=/usr/local/python3
 11. make
 12. make install
```
###### 3. 安装完成之后，将python3和pip3建立软链接，添加变量
```
 13. ln -s /usr/local/python3/bin/python3.6 /usr/bin/python3
 14. ln -s /usr/local/python3/bin/pip3.6 /usr/bin/pip3
```
###### 4. 检测python3和pip3是否正常添加到环境变量，正常返回则安装成功
```
 15. python3 
 16. pip3 -V
```
#### 第四步：安装virtualenv（方便不同版本项目管理）
```
 17. pip3 install virtualenv
 18. ln -s /usr/local/python3/bin/virtualenv /usr/bin/virtualenv
```
#### 第五步：在home目录下创建env和www，用于存放自己的项目和虚拟环境（可根据自己喜好命名）
```
 19. mkdir -p /home/env
 20. mkdir -p /home/www
```
#### 第六步：切换到/home/env/下，创建指定版本的虚拟环境，激活虚拟环境（出现(pyweb)，说明是成功进入虚拟环境。）
```
 21. cd /home/env
 22. virtualenv --python=/usr/bin/python3 pyweb
 23. cd /data/env/pyweb/bin
 24. source activate
```
#### 第七步：虚拟环境里安装Django和uwsgi（建议django安装1.11.1）
```
 25. pip3 install django==1.11.1
 26. pip3 install uwsgi
 27. ln -s /usr/local/python3/bin/uwsgi /usr/bin/uwsgi
```
#### 第八步：切换到网站目录/home/www,将自己的Django项目复制到此目录（建议scp或者rz）
```
 28. cd /data/wwwroot
 29. rz
```
#### 第九步：安装xadmin所需插件（其他所需包根据自己的项目自行安装即可）
```
 30. pip install django_import_export
 31. pip install future six httplib2
 32. pip install django-formtools
 33. pip install django-crispy-forms
```
#### 第十步：测试自己的项目是否可以正常运行（若不能正常启动，需根据错误提示安装项目所需的第三方插件，以及其他即可）
```
 34. python3 manage.py runserver 127.0.0.1:8000
```
#### 第十一步：Django正常运行之后，我们就开始配置一下uwsgi
###### 1、进入自己的项目根目录（暂且用mysite代替自己的项目名，下同），创建mysite.xml文件
```
 35. cd /home/www/mysite
 36. vim mysite.xml
```
###### 2、在mysite.xml文件中输入以下内容，按Esc键，输入 ：wq 保存退出
```
<uwsgi>
    <socket>:8000</socket><!-- 内部端口，自定义 -->
     <chdir>/home/www/mysite/</chdir><!-- 项目路径 -->
    <module>mysite.wsgi:application</module><!-- 对应项目下的wsgi.py文件 -->
     <processes>4</processes> <!-- 进程数 -->
    <daemonize>uwsgi.log</daemonize><!-- 日志文件 -->
</uwsgi>
```
#### 第十二步：安装nginx
###### 1、进入home目录，安装nginx
```
 37. cd /home
 38. wget http://nginx.org/download/nginx-1.13.7.tar.gz 
 39. tar -zxvf nginx-1.13.7.tar.gz
 40. cd nginx-1.13.7
 41. ./configure
 42. make
 43. make install
```
###### 2、进入nginx安装路径，配置nginx.conf文件（nginx一般默认安装好的路径为/usr/local/nginx）
```
 44. cd /usr/local/nginx/conf
 45. vim nginx.conf
```
###### 输入如下内容到nginx.conf，按Esc键，输入 ：wq 保存退出
```
    server {
        listen       80;
        server_name  localhost;

        charset utf-8;

        location / {
           include uwsgi_params;
           uwsgi_pass 127.0.0.1:8000;
           uwsgi_param UWSGI_SCRIPT mysite.wsgi;
           uwsgi_param UWSGI_CHDIR /home/www/mysite;
        }
        location /static/ {
            alias /home/www/mysite/statics/; #静态文件目录
        }
```
###### 3、启动nginx服务器（终端没有任何提示就证明nginx启动成功）
```
 46. cd /usr/local/nginx/sbin
 47. ./nginx -t
 48. ./nginx
```
###### 4、如若出现：[emerg] still could not bind()，查看80端口对应进程，关掉即可
```
 49. netstat -ntlp|grep 80
 50. kill -9 PID	（PID为netstat语句返回值对应ID）
```
#### 第十三步：启动uwsgi服务以及重启nginx
```
 51. cd /home/www/mysite/
 52. uwsgi -x mysite.xml
 53. cd /usr/local/nginx/sbin/
 54. ./nginx -s reload
```
#### 第十四步：在浏览器里输入服务器IP，出现期盼已久的项目，大功告成！



****

## 点点滴滴的积累都将会是你今后一笔不菲的财富，共勉