# Git代码找回

今天做了一个傻逼操作，在工作分支上还没commit就切换到dev分支，再切换回来发现代码不见了。

网上搜索了一下，在需要找回代码的分支下运行

````
git fsck --lost-found
````

然后.git文件夹会出现一个lost-found文件夹：

![image-20220605155910062](https://cdn.jsdelivr.net/gh/Brandoooon/myBlog/docs/VCS/img/image-20220605155910062.png)

用文本打开，还好都在。

不准备commit的话下次一定记得先stash。
