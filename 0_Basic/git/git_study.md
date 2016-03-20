git常用
https://www.zhihu.com/question/21995370
http://www.ruanyifeng.com/blog/2014/06/git_remote.html


Git rm – 如何使文件脱离git的版本管理，但不是会删除它

git rm --cached file

Git 将不再跟踪此文件，尽管它仍然是在您的硬盘上.





创建和remote同样的分支

git checkout -t origin/dev 该命令等同于：
git checkout -b dev origin/dev