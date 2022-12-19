# git 多用户配置
1. key gen  
    ```
    ssh-keygen -t rsa -C "name@email.com"
    ```
2. 复制公钥到仓库设置中  
    ```
    cat ~/.ssh/github.pub
    ```
3. 新建config配置多用户
    ```
    vi ~/.ssh/config
    ```
4. 填写内容
    > Host github.com  
    HostName github.com  
    User howsonliu  
    IdentityFile ~/.ssh/github  
      \
    Host gitlab.corp.youdao.com  
    HostName gitlab.corp.youdao.com  
    User liuhaosheng  
    IdentityFile ~/.ssh/gitlab

5. 删除know_hosts
    ```
    rm ~/.ssh/know_hosts
    ```
6. 验证
    ```
    ssh -T git@github.com
    ssh -T git@gitlab.corp.youdao.com
    ```