
- 生成每个账户的公钥和私钥
  ~~~
  ssh-keygen -t rsa -f ~/.ssh/user -C "user@email.com"
  ~~~
- 在.ssh目录下配置每个账户的域名(远程仓库的域名)
  建立一个config的文本文件，内容如下：

  ```html
  # 有几个账户就可以为每个账户分别配置
  Host gitlab.com
  Hostname gitlab.com
  User git
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/user
  ```

  > [!warning] 注意
  > 这里有个小坑，在直接复制内容过去的时候，使用git bash推送文件的时候会报出编码错误。如果出现的话可以在bash窗口用vim命令建立这个文件
  
- 测试链接
  使用如下命令测试连接情况
  ~~~sh
  ssh -T git@gitxx.com
  ~~~

  注意 -T大写,如果是 -t 的话对于github会抛出以下错误:

  <img src = "https://r2.129870.xyz/img/20210406233301.png" style = "float:left">

  
  切换回大写则链接成功建立,对于gitlab而言则没有这个问题:
  ![](https://r2.129870.xyz/img/20210406233526.png)
- 建立版本库，同时为每个版本库指定不同的用户
  ~~~sh
  git config --local user.name = user
  git config --local user.eamil = usere@email.com
  ~~~
- 推送到远程版本库
  ~~~
  git remote add origin git@user.github.com:xxx/xxxxx.git
  git push -u origin master
  ~~~

