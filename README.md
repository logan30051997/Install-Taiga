---
title: Huong dan cai dat taiga tren ubuntu 20.04
tags: []
---

## Hướng dẫn cài đặt Taiga trên Ubuntu 20.04
### 1.Điều kiện tiên quyết
- Cần ít nhất 1GB RAM
- Cần ít nhất 20GB bộ nhớ lưu trữ
- **Đặc biệt :** Taiga phải được cài đặt bằng user thường chứ không phải user root
- Sửa hostname
```sh
    hostnamectl set-hostname taiga.linex.vn    
```
    
### 2.Cài đặt các tool và các gói cài cần thiết cho Taiga
```sh
    sudo apt-get update -y
    sudo  apt-get install sudo -y
    sudo apt-get install vim -y
    sudo apt-get install net-tools -y
    sudo apt-get install -y build-essential binutils-doc autoconf flex bison libjpeg-dev
    sudo apt-get install -y libfreetype6-dev zlib1g-dev libzmq3-dev libgdbm-dev libncurses5-dev
    sudo apt-get install -y automake libtool curl git tmux gettext
    sudo apt-get install -y nginx
    sudo apt-get install -y rabbitmq-server
```
- Kiểm tra trạng thái và version của nginx và rabbitmq-server
```sh
    sudo systemctl status nginx
    sudo nginx -v
    sudo systemctl status rabbitmq-server
    sudo rabbitmq-diagnostics server_version
```
- Cài đặt PostgreSQL và khởi động máy chủ cơ sở dữ liệu
```sh
    sudo apt-get install -y postgresql-12 postgresql-contrib-12 postgresql-doc-12 postgresql-server-dev-12
    sudo pg_ctlcluster 12 main start
```
- Cài đặt Python3 và các thư viện của Python
```sh
    sudo apt-get install -y python3 python3-pip python3-dev python3-venv
    sudo apt-get install -y libxml2-dev libxslt-dev
    sudo apt-get install -y libssl-dev libffi-dev
```
- kiểm tra version Python
```sh
    python3 -V
```
- Cài đặt Node.js
```sh
    curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
    sudo apt-get install -y nodejs
```
- Kiểm tra version Nodejs
```sh
    node -v
```
### 3.Tạo user Taiga và cấu hình PostgreSQL & RabbitMQ
- Tạo user taiga và cấp quyền sudo cho nó
```sh
    sudo adduser taiga
    sudo adduser taiga sudo
    sudo su taiga
    cd ~
```
*Quá trình triển khai Taiga phải được hoàn tất với người dùng taiga!*

- Thay đổi mật khẩu user postgres

```sh
    sudo passwd postgres
```
-  Tạo database cho taiga ở Postgres
```sh
    sudo -u postgres createuser taiga --interactive --pwprompt
    sudo -u postgres createdb taiga -O taiga --encoding='utf-8' --locale=en_US.utf8 --template=template0
```
- Tạo người dùng RabbitMQ và một máy chủ ảo cho RabbitMQ tên là taiga
```sh
    sudo rabbitmqctl add_user rabbitmqtaiga rabbitmqtaigapassword
    sudo rabbitmqctl add_vhost taiga
    sudo rabbitmqctl set_permissions -p taiga rabbitmqtaiga ".*" ".*" ".*"
```
### 4.Thiết lập và cấu hình Backend cho Taiga
```sh
    cd ~
    git clone https://github.com/taigaio/taiga-back.git taiga-back
    cd taiga-back
    git checkout stable
-```

- Tạo một virtualenv

```
    python3 -m venv .venv --prompt taiga-back
    source .venv/bin/activate
    (taiga-back) pip install --upgrade pip wheel
```

- Cài đặt tất cả các gói phụ thuộc cho Python

```sh
    (taiga-back) pip install -r requirements.txt
```

- Cài đặt taiga-Contrib-Protection

```sh
    (taiga-back) pip install git+https://github.com/taigaio/taiga-contrib-protected.git@6.0.0#egg=taiga-contrib-protected
```
- Copy và chỉnh sửa file cài đặt cấu hình settings/config.py,bên dưới nó càng dòng chú ý cần sửa
```sh
    cp settings/config.py.prod.example settings/config.py

```
```sh
    vi settings/config.py
```
```sh
   DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'taiga',
        'USER': 'taiga',
        'PASSWORD': 'Anhduong1908',
        'HOST': '',
        'PORT': '',
    }
} 
    SECRET_KEY = "linex@2021"
    TAIGA_URL = "https://taiga.linex.vn"
    SITES = {
    "api": {"domain": "taiga.linex.vn", "scheme": "https", "name": "api"},
    "front": {"domain": "taiga.linex.vn", "scheme": "https", "name": "front"}
    ## EVENTS
#########################################
    EVENTS_PUSH_BACKEND = "taiga.events.backends.rabbitmq.EventsPushBackend"
    EVENTS_PUSH_BACKEND_OPTIONS = {
    "url": "amqp://duonglinex:Anhduong1908@localhost:5672/taiga"
}
## TAIGA ASYNC
#########################################
    CELERY_ENABLED = os.getenv('CELERY_ENABLED', 'True') == 'True'

from kombu import Queue  # noqa

    CELERY_BROKER_URL = "amqp://duonglinex:Anhduong1908@localhost:5672/taiga"
}
    CELERY_TIMEZONE = 'Asia/Ho_Chi_Minh'
    
```
- Thực thi tất cả các lệnh dưới để điền vào cơ sở dữ liệu những dữ liệu cần thiết cơ bản ban đầu:
```sh
   source .venv/bin/activate
    (taiga-back) DJANGO_SETTINGS_MODULE=settings.config python manage.py migrate --noinput
# create an administrator with strong password
    (taiga-back) CELERY_ENABLED=False DJANGO_SETTINGS_MODULE=settings.config python manage.py createsuperuser
    (taiga-back) DJANGO_SETTINGS_MODULE=settings.config python manage.py loaddata initial_project_templates
    (taiga-back) DJANGO_SETTINGS_MODULE=settings.config python manage.py compilemessages
    (taiga-back) DJANGO_SETTINGS_MODULE=settings.config python manage.py collectstatic --noinput 
```
### 5.Thiết lập và cấu hình Fontend cho Taiga
```sh
    cd ~
    git clone https://github.com/taigaio/taiga-front-dist.git taiga-front-dist
    cd taiga-front-dist
    git checkout stable
```
- Sao chép tệp cấu hình và chỉnh sửa như bên dưới
```sh
    sudo cp ~/taiga-front-dist/dist/conf.example.json ~/taiga-front-dist/dist/conf.json
```
```sh
    sudo vim ~/taiga-front-dist/dist/conf.json
```
```sh
    {
    "api": "https://taiga.linex.vn/api/v1/",
    "eventsUrl": "wss://taiga.linex.vn/events",
    "eventsMaxMissedHeartbeats": 5,
    "eventsHeartbeatIntervalTime": 60000,
    "eventsReconnectTryInterval": 10000,
    "debug": true,
    "debugInfo": false,
    "defaultLanguage": "en",
    "themes": ["taiga"],
    "defaultTheme": "taiga",
    "defaultLoginEnabled": true,
    "publicRegisterEnabled": true,
    "feedbackEnabled": true,
    "supportUrl": "https://resources.taiga.io",
    "privacyPolicyUrl": null,
    "termsOfServiceUrl": null,
    "maxUploadFileSize": null,
    "contribPlugins": [],
    "tagManager": { "accountId": null },
    "tribeHost": null,
    "enableAsanaImporter": false,
    "enableGithubImporter": false,
    "enableJiraImporter": false,
    "enableTrelloImporter": false,
    "gravatar": false,
    "rtlLanguages": ["ar", "fa", "he"]
}
```
### 6.Thiết lập Taiga Events
```sh
    cd ~
    git clone https://github.com/taigaio/taiga-events.git taiga-events
    cd taiga-events
    git checkout stable
```
- Cài đặt các phụ thuộc JavaScript bắt buộc:
```sh
    npm install
```
- Tạo .envtệp dựa trên mẫu được cung cấp và chỉnh sửa nó
```sh
    cp .env.example .env
    vim .env
```
```sh
    RABBITMQ_URL="amqp://duonglinex:Anhduong1908@localhost:5672"
    SECRET="linex@2021"
    WEB_SOCKET_SERVER_PORT=8888
    APP_PORT=3023
```
### 7.Thiết lập Taiga Protection
```sh
    cd ~
    git clone https://github.com/taigaio/taiga-protected.git taiga-protected
    cd taiga-protected
    git checkout stable
```
- Tạo một Virtualenv
```sh
    python3 -m venv .venv --prompt taiga-protected
    source .venv/bin/activate
    (taiga-protected) pip install --upgrade pip wheel
```
- Cài đặt tất cả các phụ thuộc Python
```sh
    (taiga-protected) pip install -r requirements.txt
```
- Sao chép tệp cấu hình ví dụ:
```sh
    cp ~/taiga-protected/env.sample ~/taiga-protected/.env
```
```sh
    SECRET_KEY=linex@2021
    MAX_AGE=300
```
### 8.Triển khai chạy các dịch vụ Taiga tương ứng
- Tạo một tệp systemd mới tại  /etc/systemd/system/taiga.service để chạy taiga-back :
```sh
   sudo vim /etc/systemd/system/taiga.service 
```
```sh
    [Unit]
    Description=taiga_back
    After=network.target

    [Service]
    User=taiga
    WorkingDirectory=/home/taiga/taiga-back
    ExecStart=/home/taiga/taiga-back/.venv/bin/gunicorn --workers 4 --timeout 60 --log-level=info --access-logfile - --bind 0.0.0.0:8001 taiga.wsgi
    Restart=always
    RestartSec=3

    Environment=PYTHONUNBUFFERED=true
        Environment=DJANGO_SETTINGS_MODULE=settings.config

    [Install]
    WantedBy=default.target
```
- Tải lại daemon systemd và khởi động dịch vụ taiga:
```sh
    sudo systemctl daemon-reload
    sudo systemctl start taiga
    sudo systemctl enable taiga
```
- Tạo một tệp systemd mới tại /etc/systemd/system/taiga-async.service để chạy taiga-async :
```sh
    sudo vim /etc/systemd/system/taiga-async.service
```
```sh
    [Unit]
    Description=taiga_async
    After=network.target

    [Service]
    User=taiga
    WorkingDirectory=/home/taiga/taiga-back
    ExecStart=/home/taiga/taiga-back/.venv/bin/celery -A taiga.celery worker -B --concurrency 4 -l INFO
    Restart=always
    RestartSec=3
    ExecStop=/bin/kill -s TERM $MAINPID

    Environment=PYTHONUNBUFFERED=true
    Environment=DJANGO_SETTINGS_MODULE=settings.config

    [Install]
    WantedBy=default.target
```
-  Tải lại daemon systemd và khởi động dịch vụ taiga-async :
```sh
    sudo systemctl daemon-reload
    sudo systemctl start taiga-async
    sudo systemctl enable taiga-async
```
- Tạo một tệp systemd mới tại /etc/systemd/system/taiga-events.service để chạy taiga-event :
```sh
    sudo vim /etc/systemd/system/taiga-events.service
```

```sh
    [Unit]
    Description=taiga_events
    After=network.target

    [Service]
    User=taiga
    WorkingDirectory=/home/taiga/taiga-events
    ExecStart=npm run start:production
    Restart=always
    RestartSec=3

    [Install]
    WantedBy=default.target
```
- Tải lại daemon systemd và khởi động dịch vụ taiga-events:
```sh
    sudo systemctl daemon-reload
    sudo systemctl start taiga-events
    sudo systemctl enable taiga-events
```
- Tạo một tệp systemd mới tại /etc/systemd/system/taiga-protected.service để chạy được taiga-protected :
```sh
    sudo vim /etc/systemd/system/taiga-protected.service
```
```sh
    [Unit]
    Description=taiga_protected
    After=network.target

    [Service]
    User=taiga
    WorkingDirectory=/home/taiga/taiga-protected
    ExecStart=/home/taiga/taiga-protected/.venv/bin/gunicorn --workers 4 --timeout 60 --log-level=info --access-logfile - --bind 0.0.0.0:8003 server:app
    Restart=always
    RestartSec=3

    Environment=PYTHONUNBUFFERED=true

    [Install]
    WantedBy=default.target
```
- Tải lại daemon systemd và khởi động dịch vụ  taiga-protected
```sh
    sudo systemctl daemon-reload
    sudo systemctl start taiga-protected
    sudo systemctl enable taiga-protected
```
- Xóa tệp cấu hình NGINX mặc định để tránh va chạm với Taiga:
```sh
    sudo rm /etc/nginx/sites-enabled/default
```
```sh
    mkdir -p ~/logs
```
### 9.Cấu hình Nginx và tiến hành chạy thử
- Để định cấu hình máy chủ ảo NGINX mới cho Taiga, hãy tạo và chỉnh sửa /etc/nginx/conf.d/taiga.conf tệp, như sau:
```sh
    sudo vim /etc/nginx/conf.d/taiga.conf
```
```sh
    server {
    listen 80 ;
    server_name taiga.linex.vn;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl
    server_name taiga.linex.vn;
    ssl_certificate_key /etc/ssl/mykeys/taiga-ssl.local.key;
    ssl_certificate /etc/ssl/mykeys/taiga-ssl.local.crt;

    large_client_header_buffers 4 32k;
    client_max_body_size 50M;
    charset utf-8;

    access_log /home/taiga/logs/nginx.access.log;
    error_log /home/taiga/logs/nginx.error.log;

    # Frontend
    location / {
        root /home/taiga/taiga-front-dist/dist/;
        try_files $uri $uri/ /index.html;
    }

    # Backend
    location /api {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://127.0.0.1:8001/api;
        proxy_redirect off;
    }

    # Admin access (/admin/)
    location /admin {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://127.0.0.1:8001$request_uri;
        proxy_redirect off;
    }

    # Static files
    location /static {
        alias /home/taiga/taiga-back/static;
    }

    # Media
    location /_protected {
        internal;
        alias /home/taiga/taiga-back/media/;
        add_header Content-disposition "attachment";
    }

    # Unprotected section
    location /media/exports {
        alias /home/taiga/taiga-back/media/exports/;
        add_header Content-disposition "attachment";
    }

    location /media {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://127.0.0.1:8003/;
        proxy_redirect off;
    }

    # Events
    location /events {
        proxy_pass http://127.0.0.1:8888/events;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_connect_timeout 7d;
        proxy_send_timeout 7d;
        proxy_read_timeout 7d;
    }
}
```
- Thực thi lệnh sau để xác minh cấu hình NGINX và theo dõi bất kỳ lỗi nào trong dịch vụ:
```sh
    sudo nginx -t
```
- Cuối cùng khởi động lại dịch vụ nginx và các dịch vụ của taiga
```sh
    sudo systemctl restart nginx
    sudo systemctl restart 'taiga*'
```
- Mở trình duyệt lên và gõ https://taiga.linex.vn
![](image-kmk98gq6.png)
- Quá trình cài đặt thành công ,ta chọn login và tiến hành đăng nhập super user
![](image-kmk99cst.png)
- Giao diện quản lý đã sẵn sàng để làm việc
![](image-kmk9a49a.png)
