# docker_gitlab
- [gcp 創建 instance 操作範例](#gcp-創建-instance-操作範例)
  - [創建執行個體](#創建執行個體)
  - [設定名稱](#設定名稱)
  - [選擇機器等級](#選擇機器等級)
  - [作業系統設定](#作業系統設定)
  - [公開ip設定](#公開ip設定)
- [ssh 連線設定](#ssh-連線設定)
  - [瀏覽器連線](#瀏覽器連線)
  - [連線設定](#連線設定)
- [docker與docker-compose安裝](#docker與docker-compose安裝)
- [自建docker gitlab](#自建docker-gitlab)
  - [clone專案](#clone專案)
  - [啟動服務](#啟動服務)
  - [修改域名](#修改域名)
  - [首次登入與新增使用者](#首次登入與新增使用者)


## gcp 創建 instance 操作範例

### 創建執行個體

<img width="1388" alt="創立執行個體" src="https://github.com/JackViewLand/docker_gitlab/assets/122655131/c486599d-e908-493e-adc6-af3c51530a0c">

### 設定名稱

<img width="1341" alt="設定名稱" src="https://github.com/JackViewLand/docker_gitlab/assets/122655131/30e3e3d1-544a-4313-936d-7c17ab9fcc9b">

### 選擇機器等級

<img width="661" alt="選擇機器等級" src="https://github.com/JackViewLand/docker_gitlab/assets/122655131/f2acad10-5b0b-41e9-b47d-fb1e02025755">

### 作業系統設定

<img width="655" alt="變更OS" src="https://github.com/JackViewLand/docker_gitlab/assets/122655131/a86bd822-a5ee-4632-bd65-4fc2109d7ed1">
<img width="635" alt="選擇OS" src="https://github.com/JackViewLand/docker_gitlab/assets/122655131/afa0db67-c6da-44f5-86e1-fd8dc3187dfb">

### 公開ip設定

IP預設是```隨機分配```的，```重新開機後依然會隨機分配一個新的public IP```，如果不希望每次重開就調整DNS，想讓機器擁有固定的IP，可以依照以下操作設定。

<img width="600" alt="進階選項" src="https://github.com/JackViewLand/docker_gitlab/assets/122655131/684d475d-9e89-4987-8ad9-12d64eb5f4b7">
<img width="579" alt="進階設定IP" src="https://github.com/JackViewLand/docker_gitlab/assets/122655131/010e018e-1ad9-4a13-bf32-be2532c3d389">
<img width="515" alt="IP設定" src="https://github.com/JackViewLand/docker_gitlab/assets/122655131/fbd7144f-b87a-47bb-9a38-791cbaea317d">

------------------
## ssh 連線設定
### 瀏覽器連線
<img width="880" alt="瀏覽器連線" src="https://github.com/JackViewLand/docker_gitlab/assets/122655131/6596cc5d-f642-429a-afb2-c17a6f5dec7b">

### 連線設定
* 如果我在預設帳號新增ssh key 重啟後設定會被刷掉
* 先在```本地mac```生成ssh key
  ```bash
  ssh-keygen
  ```
  記得把key權限改為600
  ```bash
  chmod 600 id_rsa
  ```
* 在```gcp```新增使用者並加入ssh 連線
  * 先在root新增使用者 jack
    ```bash
    sudo adduser jack
    ```
  * 切換使用者jack
    ```bash
    su jack
    ```
  * 新增ssh設定
    ```bash
    mkdir ~/.ssh
    ```
    ```bash
    vi ~/.ssh/authorized_keys
    # 請貼上本地生成的.pub key
    ```
  * 新增使用者與root權限切換
    ```bash
    # 切回root
    # 新增 /etc/sudoers write權限
    chmod u+w /etc/sudoers
    # 將新使用者設定切換root權限
    vi /etc/sudoers
    # 找到 root ALL =(ALL) ALL，並在下面新增其中一行
    # 允許用戶執行sudo命令(需要輸入密碼) 
    <YOURUSER> ALL =(ALL) ALL 
    # 允許用戶組執行sudo命令(不需要輸入密碼) 
    %<YOURUSER> ALL =(ALL) ALL
    # 允許用戶執行sudo命令(需要輸入密碼) 
    <YOURUSER> ALL =(ALL) NOPASSWD:ALL
    # 允許用戶組執行sudo命令(不需要輸入密碼) 
    %<YOURUSER> ALL =(ALL) NOPASSWD:ALL
    ```
* ```mac本地```設定ssh alias
    ```bash
    vi ~/.ssh/config
    ```
    ```bash
    Host jack-server
        HostName <PUBLIC IP>
        User <USERNAME>
        Port 22
        IdentityFile /YOUR/PATH/.ssh/id_rsa
    ```
    
------------------

## docker與docker-compose安裝
* 更新apt套件
  ```bash
  sudo apt-get update
  ```
  
  ```bash
  sudo apt-get -y install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
  ```
* 添加docker官方GPG密鑰
  ```bash
  sudo mkdir -p /etc/apt/keyrings
  ```
  ```bash
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
  ```
* 添加docker的apt儲存庫
  ```bash
  echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  ```
* 安裝Docker與Docker compose
  ```bash
  sudo apt-get update
  ```
  ```bash
  sudo apt-get -y install docker-ce docker-ce-cli containerd.io docker-compose-plugin
  ```
  ```bash
  sudo ln -s /usr/libexec/docker/cli-plugins/docker-compose /usr/bin/docker-compose
  ```
* 新增使用者權限

  新增群組docker
  
  ```bash
  sudo groupadd docker
  ```

  將指定帳戶加入群組

  ```bash
  sudo usermod -aG docker <username>
  ```

  更新用戶組

  ```bash
  newgrp docker
  ```

  ```最後退出gcp重新登入確認docker是否安裝成功```

------------------
## 自建docker gitlab

  ### clone專案
  
  ```bash
  git clone https://github.com/JackViewLand/docker_gitlab.git
  ```
  ### 啟動服務 
  ```bash
  docker-compose up -d 
  ```
  
  ### 修改域名
  ```bash
  sudo vi config/gitlab.rb
  ```
  <img width="688" alt="ggitlab" src="https://github.com/JackViewLand/docker_gitlab/assets/122655131/80ee1ce2-ab11-4758-ba93-49cdbbea74b5">
  
  重新啟動
  
  ```bash
  docker-compose down -v && docker-compose up -d 
  ```
  ### 首次登入與新增使用者
  
  <img width="1000" alt="login" src="https://github.com/JackViewLand/docker_gitlab/assets/122655131/a2f7ecda-150e-4bf4-9c44-054a15914f03">
  
  第一次登入root密碼存放在```config/initial_root_password```
  
  ```bash
  sudo cat config/initial_root_password
  ```
  
  <img width="577" alt="截圖 2024-01-20 下午2 24 51" src="https://github.com/JackViewLand/docker_gitlab/assets/122655131/9c18ee39-2fce-42f2-8004-d044a66f5327">
  
  登入成功後請先修改root密碼
  
  <img width="258" alt="修改root密碼" src="https://github.com/JackViewLand/docker_gitlab/assets/122655131/f5095feb-785f-46e1-b904-ff8d13b476fe">
  <img width="993" alt="截圖 2024-01-20 下午2 30 42" src="https://github.com/JackViewLand/docker_gitlab/assets/122655131/169f90d8-c029-4eb9-a43a-b4e26c812053">

  新增normal user
  
  <img width="300" alt="新增normal_user" src="https://github.com/JackViewLand/docker_gitlab/assets/122655131/55ab2de4-0455-462d-94ec-9b78129c5c61">

  <img width="257" alt="user" src="https://github.com/JackViewLand/docker_gitlab/assets/122655131/cf5a4b04-b8f9-4837-9119-70085d69e985">

  # 恭喜你成功搭建私有的gitlab囉！

