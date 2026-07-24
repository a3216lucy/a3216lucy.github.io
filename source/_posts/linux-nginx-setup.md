---
title: Linux 裸機安裝 Nginx 部署前端筆記
date: 2025-02-26 11:38:00
categories: 
- 技術筆記
tags:
- Linux
- Nginx
---

紀錄一下在 Linux 裸機環境從零安裝 Nginx、部署前端靜態網站，到設定 HTTPS 的完整流程。

<!-- more -->

## 1. 安裝 Nginx

首先，安裝 Nginx：

```shell
sudo apt update
sudo apt install nginx
```

## 2. 建立目錄

接著，建立你所需要的目錄 `/var/www/html/front`，並確保 Nginx 有權限讀取該目錄：

```shell
sudo mkdir -p /var/www/html/front
```

## 3. 設定目錄權限

確保 Nginx 能夠讀取該目錄，將權限設置為 Nginx 的執行使用者（默認是 www-data）：

```shell
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html
```

## 4. 修改 Nginx 設定

打開 Nginx 設定檔（`/etc/nginx/sites-available/default`），修改 root 設定，使其指向你創建的目錄 `/var/www/html/front`：

```shell
sudo vim /etc/nginx/sites-available/default
```

修改 server 區塊中的 root 設定為：

```ts
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html/front;
    index index.html;

    server_name _;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

## 5. 測試 Nginx 設定

在重新啟動 Nginx 之前，先檢查 Nginx 配置是否正確：

```shell
sudo nginx -t
```

如果沒有錯誤，將顯示 syntax is okay 和 test is successful。

## 6. 重新啟動 Nginx

如果配置沒有問題，重新啟動 Nginx 使設定生效：

```shell
sudo systemctl restart nginx
```

## 7. 上傳前端檔案

將你的前端 build 檔案（通常在 dist 或 build 資料夾內）複製到 `/var/www/html/front` 目錄：

```shell
scp -r ./dist/* user@your_server:/var/www/html/front/
```

或者如果你在伺服器內操作，直接複製：

```shell
cp -r ./dist/* /var/www/html/front/
```

---

若是發生 scp 時，permission denied，則可以授權登入的帳號擁有該目錄，然後再重新執行一次 scp。

```shell
sudo chown -R your_user:your_user /var/www/html/front
```

---
若想刪除，可以使用：

```shell
sudo rm -rf /var/www/html/front/*
```

## 8. 測試前端佈署

完成後，先確認 Nginx 是否有在運行：

```shell
sudo systemctl status nginx
```

如果看到 **“active (running)”**，代表 Nginx 正在運行。
如果沒有運行，可以啟動：

```shell
sudo systemctl start nginx
```

再來，確認 Nginx 監聽的 Port

```shell
sudo netstat -tulnp | grep nginx
```

會看到類似

```shell
tcp   LISTEN  0  128  0.0.0.0:80   0.0.0.0:*   users:(("nginx",pid=1234,fd=6))
```

這表示 Nginx 監聽 80 Port
可以在瀏覽器中輸入 server IP:port 來檢查前端是否正確顯示，例如：

```md
http://your_server_IP:port
```

## 補充一：調整 Nginx 設定檔

預設安裝，通常 Nginx 會監聽 80。
但你可以檢查設定：

```shell
sudo vim /etc/nginx/sites-available/default

// 若有多個站台，也會各自有一份設定檔
sudo vim /etc/nginx/sites-available/example.com
```

看看 server 區塊：

```ts
server {
    listen 80;
    server_name YOUR_SERVER_IP;

    root /var/www/html/front;
    index index.html;

    location / {
        try_files $uri /index.html;
    }
}
```

- listen 80; → 代表 Nginx 監聽 80 Port
- root /var/www/html/front; → 你的前端 build 檔案應該在這個目錄
- index index.html; → 預設加載 index.html

如果修改了設定，記得重新啟動 Nginx：

```shell
sudo systemctl restart nginx
```

這樣就完成了！

## 補充二：修改 Nginx 樣式

若是發現 assets、css style 有讀不到問題，可以至 nginx.conf 調整：

```shell
 sudo vim  /etc/nginx/nginx.conf
```

完成配置修改後，檢查 Nginx 配置是否正確：

```shell
sudo nginx -t
```

如果配置文件沒有錯誤，重新啟動 Nginx 使更改生效：

```shell
sudo systemctl restart nginx
```

這樣就完成了！

## 補充三：查看錯誤 Log

可以執行以下指令來觀看：

```shell
// 錯誤 LOG
sudo tail -f /var/log/nginx/error.log

// 執行 API LOG
sudo cat /var/log/nginx/access.log
```

## 補充四：預設讓 http（80 port） 都導轉至 https（443 port）

> 參考文章：
>  - [用Nginx架設local端https應用](https://hackmd.io/@warrenpig/create-self-signed-https-nginx-app)
>  - [NGINX 設定 HTTPS 網頁加密連線，建立自行簽署的 SSL 憑證](https://blog.gtwang.org/linux/nginx-create-and-install-ssl-certificate-on-ubuntu-linux/)

### 建立一個放置憑證的目錄

目錄的路徑可以自由選擇，放在哪裡都可以，而考量到這個憑證是 NGINX 專用的，所以跟 NGINX 的設定檔放在一起可能會比較好管理。

```shell
sudo mkdir /etc/nginx/ssl
```

### 產生自行簽署的 SSL 憑證

使用 `openssl` 產生自行簽署的 SSL 憑證，並且將憑證的存放路徑設成剛剛上面建立的目錄：

```shell
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/nginx.key -out /etc/nginx/ssl/nginx.crt
```

若是要 .p12 檔 轉成 .crt 跟 .key，可以使用：

```shell
sudo openssl pkcs12 -in /etc/ssl/private/your_cert.p12 -clcerts -nokeys -out /etc/ssl/private/your_cert.crt -legacy

sudo openssl pkcs12 -in /etc/ssl/private/your_cert.p12 -nocerts -nodes -out /etc/ssl/private/your_cert.key -legacy
```

以下是這裡使用到的參數與簡略說明：

- `req`：使用 X.509 Certificate Signing Request（CSR） Management 產生憑證。
- `-x509`：建立自行簽署的憑證。
- `-nodes`：不要使用密碼保護，因為這個憑證是 NGINX 伺服器要使用的，如果設定密碼的話，會讓伺服器每次在啟動時都需要輸入密碼。
- `-days 365`：設定憑證的使用期限，單位是天，如果不想時常重新產生憑證，可以設長一點。
- `-newkey rsa:2048`：同時產生新的 RSA 2048 位元的金鑰。
- `-keyout`：設定金鑰儲存的位置。
- `-out`：設定憑證儲存的位置。

這裡我們會同時建立憑證與金鑰，建立的過程中會需要填寫一些基本的資料：

- Country Name：TW
- State or Province Name (full name)：Taiwan
- Locality Name (eg, city)：Taipei
- Organization Name (eg, company)：My Company
- Organizational Unit Name (eg, section)：My Unit
- Common Name (e.g. server FQDN or YOUR name)：example.com
- Email Address：you@example.com

1. 國家代碼，台灣就填 `TW`。
2. 州或省，台灣就填 `Taiwan`。
3. 城市，例如台北就填 `Taipei`。
4. 公司名稱。
5. 部門名稱。
6. 伺服器的 FQDN，這個一定要填寫正確，如果沒有申請網域名稱的話，也可以用 IP 位址替代。
7. E-mail 信箱。

填寫完成之後，憑證與金鑰的建立就完成了，而存放位置就在 `/etc/nginx/ssl` 目錄中。

### 設定 NGINX 伺服器

在原本的設定檔中

```shell
sudo vim /etc/nginx/sites-available/default
```

加上 SSL 的設定，並且設定憑證與金鑰的路徑：

```json
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    # 導向至 HTTPS
    rewrite ^(.*) https://$host$1 permanent;
}

server {
    # 加入 SSL 設定
    listen 443 ssl default_server;
    listen [::]:443 ssl default_server;

    # 憑證與金鑰的路徑
    ssl_certificate /etc/nginx/ssl/nginx.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx.key;    

    root /var/www/html/front;
    index index.html index.htm index.nginx-debian.html;
    server_name _;
    
    location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.
        try_files $uri $uri/ =404;
    }

    # 反向代理後端 API 的範例
    location /api/ {
        proxy_pass https://127.0.0.1:5001/api/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;  # 設置轉發的協議為 HTTPS
    }
}
```

接著重新啟動 NGINX 伺服器：

```shell
sudo service nginx restart
```
