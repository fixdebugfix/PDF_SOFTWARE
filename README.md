# PrivateBlog

主要用于记录自己的写的一些笔记和保存一些资料，这个仓库永远`private`



下面是2篇-git和github配置的文章

- [[ github \] github clone private repo 克隆私有项目 详细 - 简书 (jianshu.com)](https://www.jianshu.com/p/0503722f69af)

- [github 配置使用 personal access token 认证 - 简书 (jianshu.com)](https://www.jianshu.com/p/ab5977d763e9)


git lfs处理>25M文件
```ini
cd upload #进入名为upload的文件夹，提前将要上传的大文件放入该文件夹下
git init #创建本地仓库环境
git lfs install #安装大文件上传应用
git lfs track * #追踪要上传的大文件，*表示路径下的所有文件
git add .gitattributes #添加先上传的属性文件(要先上传属性文件，不然有可能失败)
git commit -m "pre" #添加属性文件上传的说明
git remote add origin https://github.com/xxx.git #建立本地和Github仓库的链接
git push origin master #上传属性文件
git add * #添加要上传的大文件，*表示路径下的所有文件
git commit -m "Git LFS commit" #添加大文件上传的说明
git push origin master #上传大文件

git lfs：
[https://support.huaweicloud.com/usermanual-codehub/devcloud_hlp_0960.html#devcloud_hlp_0960__section286116283444](https://support.huaweicloud.com/usermanual-codehub/devcloud_hlp_0960.html#devcloud_hlp_0960__section286116283444)

```
