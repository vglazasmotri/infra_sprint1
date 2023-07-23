# Kittygram
Блог для создания, редактирования и обмена фотографиями котиков.

## Технологии
- Python 3.x
- node.js 9.x.x
- frontend: React
- backend: Django
- nginx
- gunicorn

# Установка из репозитория

## Клонирование проекта с GitHub на сервер
```
git clone git@github.com:vglazasmotri/infra_sprint1.git
```

## Перейти в директорию с клонированным репозиторием, создать и активировать виртуальное окружение
```
cd infra_sprint1/backend/
```
```
python3 -m venv venv
```
```
source venv/bin/activate
```

## Установить зависимости для Python
```
pip install -r ../requirements.txt
```

## Выполнить миграции
```
python manage.py migrate
```

## Создать суперюзера
```
python manage.py createsuperuser
```

## В файле infra_sprint1/backend/kittygram_backend/settings.py добавьте в список ALLOWED_HOSTS внешний IP сервера, '127.0.0.1', localhost и домен:
```
nano settings.py
```
```
ALLOWED_HOSTS = ['xxx.xxx.xxx.xxx', '127.0.0.1', 'localhost', 'доменное_имя']
```

## В этом же файле поменять значение переменной DEBUG с True на False и добавить ссылку на SECRET_KEY
```
DEBUG = False
```
```
from dotenv import load_dotenv

load_dotenv()

SECRET_KEY = os.getenv('SECRET_KEY')
```

## В папке с проектом создайт файл .env с SECRET_KEY
```
SECRET_KEY=значение_ключа
```

## В settings.py поменять значение переменной STATIC_URL и добавить новую STATIC_ROOT
```
STATIC_URL = 'static_backend'
STATIC_ROOT = BASE_DIR / 'static_backend' 
```

## Собрать статику для админки Django 
```
python manage.py collectstatic
```

## Чтобы сделать файлы статики бэкенда доступными Nginx, cкопируйте директорию static_backend/ в директорию /var/www/kittygram/</i><br>
```
sudo cp -r ./static_backend/ /var/www/kittygram/
```

## Установить node.js
```
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\
sudo apt-get install -y nodejs
```

## Перейти в infra_sprint1/frontend и установить зависимости для frontend-приложения
```
npm i
```

## Из директории с фронтенд-приложением выполните команду:
```
npm run build
```

## Скопируйте статику фронтенд-приложения в директорию по умолчанию:
```
sudo cp -r build/. /var/www/kittygram/
```

## Для backend-приложения установить WSGI-сервер gunicorn
```
pip install gunicorn==20.1.0
```

## Проверьте его работу, перейдите в директорию с файлом manage.py, и запустите Gunicorn
```
gunicorn --bind 0.0.0.0:8080 kittygram_backend.wsgi
``` 

## Создать файл конфигурации для автозапуска WSGi-сервера
```
sudo nano /etc/systemd/system/gunicorn_kittygram.service
```

## Внести в него следующие настройки
```
[Unit]
Description=gunicorn daemon 
After=network.target 

[Service]
User=<имя-пользователя-в-системе>  # (!) заменить на собственное
WorkingDirectory=/home/<имя-пользователя>/infra_sprint1/backend/
ExecStart=/home/<имя-пользователя>/infra_sprint1/backend/venv/bin/gunicorn --bind 0.0.0.0:8080 kittygram_backend.wsgi

[Install]
WantedBy=multi-user.target 
```

## Запустить созданную службу и внести её в автозапуск
```
sudo systemctl start gunicorn_kittygram
```
```
sudo systemctl enable gunicorn_kittygram
```

## Проверяем запустился ли gunicorn_kittygram
```
sudo systemctl status gunicorn_kittygram
```

## Установить веб-сервер и запустить его
```
sudo apt install nginx -y
```
```
sudo systemctl start nginx 
```

## Открыть порты для фаерволла и активировать его
```
sudo ufw allow 'Nginx Full'
```
```
sudo ufw allow OpenSSH
```
```
sudo ufw enable
```

## Создать папку для медиафайлов в директории веб-сервера, изменить права доступа
```
cd /var/www/kittygram/
```
```
mkdir media
```
```
sudo chown -R <имя_пользователя> /var/www/kittygram/media/
```


## В файле настроек infra_sprint1/backend/kittygram_backend/settings.py значения MEDIA_URL и MEDIA_ROOT должны быть указаны так.
```
MEDIA_URL = '/media/'
MEDIA_ROOT = '/var/www/kittygram/media'
```

## Перейти в файл конфигурации веб-сервера и изменить его настройки на следующие (для доступа нужны права root)
```
sudo nano /etc/nginx/sites-enabled/default
```
```
server {

    server_name kittygram.servequake.com;

    location /api/ {
        proxy_pass http://127.0.0.1:8080;
        client_max_body_size 20M;
    }

    location /admin/ {
        proxy_pass http://127.0.0.1:8080;
        client_max_body_size 20M;
    }

    location /media/ {
        alias /var/www/kittygram/media/;
    }

    location / {
    root    /var/www/kittygram;
    index   index.html index.htm;
    try_files  $uri /index.html;
    }

}
```

## Проверить файл конфигурации веб-сервера, перезагрузить его конфиг в случае успеха
```
sudo nginx -t
```
```
sudo systemctl reload nginx
```

## Получить SSL-сертификат для вашего доменного имени при помощи certbot
```
sudo apt install snapd
```
```
sudo snap install core; sudo snap refresh core
```
```
sudo snap install --classic certbot
```
```
sudo ln -s /snap/bin/certbot /usr/bin/certbot 
```
```
sudo certbot --nginx
```

## Авторы
- Команда Yandex Practicum
- DevOps Сергей Сыч
