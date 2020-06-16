# 常用命令

```shell
# 安装 git
yum -y install git
```

| 命令                                 | 说明                                             |
| :----------------------------------- | :----------------------------------------------- |
| git --version                        | 查看版本                                         |
| git status                           | 查看状态                                         |
| git log                              | 查看日志                                         |
| git rm -rf --cached .                | 清除本地缓存                                     |
| git init --bare xxxr.git             | 创建 git 私有库                                  |
| git config user.name                 | 查看用户名                                       |
| git config user.email                | 查看邮箱地址                                     |
| git config --global user.name "xxx"  | 修改用户名                                       |
| git config --global user.email "xxx" | 修改邮箱地址                                     |
| git clone                            | 克隆代码库                                       |
| git pull                             | 更新本地代码                                     |
| git add                              | 添加新的代码文件                                 |
| git commit                           | 提交, 选择需要提交的文件, 添加注释；             |
| git push                             | 本地提交代码，推到git服务器                      |
| git reset --hard HEAD^               | 回退到上个版本                                   |
| git reset --hard HEAD~3              | 回退到前3次提交之前，以此类推，回退到n次提交之前 |
| git reset --hard commit_id           | 退到/进到 指定 commit 的 sha 码                  |
| git push origin HEAD --force         | 强推到远程                                       |