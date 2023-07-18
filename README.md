# Kittygram — социальная сеть для обмена фотографиями любимых питомцев.

## Описание проекта
Пользователи могут регистрироваться, загружать фотографии с описанием и смотреть питомцев других пользователей.

## Технологии

 - Python 3.9
 - Django==3.2.3
 - djangorestframework==3.12.4
 - Nginx
 - gunicorn

## Установка
<i>Примечание: Все примеры указаны для Linux</i><br>
<br>1. Клонировать репозиторий::
```
git clone git@github.com:1emd/infra_sprint1.git
```

<br>2. Создать и активировать виртуальное окружение:
```
python3 -m venv venv
```
```
source venv/bin/activate
```

<br>3. Установить зависимости для Python:
```
pip install -r requirements.txt 
```

<br>4. Перейти в папку backend и выполнить миграции:
```
cd infra_sprint1/backend/
```
```
python manage.py migrate
```

<br>5. Создать суперпользователя:
```
python manage.py createsuperuser
```

<br>6. Отредактировать файл infra_sprint1/backend/kittygram_backend/settings.py и добавить локальные адреса и доменное имя в переменную ALLOWED_HOSTS:
```
ALLOWED_HOSTS = ['127.0.0.1', '0.0.0.0', 'localhost', 'xxx.xxx.xxx.xxx']
```

<br>7. В том же файле изменить значение переменной DEBUG с True на False:
```
DEBUG = False
```

<br>8. Установить node.js:
```
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\
sudo apt-get install -y nodejs
```

<br>9. Перейти в infra_sprint1/frontend и установить зависимости для frontend-приложения:
```
npm i
```

<br>10. Установить WSGI-сервер Gunicorn для backend-приложения:
```
pip install gunicorn
```

<br>11. Создать файл конфигурации для автозапуска WSGi-сервера:
```
sudo nano /etc/systemd/system/gunicorn.service
```

<br>12. Добавить следующую конфигурацию в созданный файл:
```
[Unit]
Description=gunicorn daemon 
After=network.target 

[Service]
User=<имя-пользователя-в-системе>  # (!) заменить на собственное
WorkingDirectory=/home/<имя-пользователя>/infra_sprint1/backend/
ExecStart=/home/<имя-пользователя>/infra_sprint1/venv/bin/gunicorn --bind 0.0.0.0:8000 backend.wsgi

[Install]
WantedBy=multi-user.target 
```

<br>13. Запустить службу Gunicorn и включить ее в автозапуск:
```
sudo systemctl start gunicorn
```
```
sudo systemctl enable gunicorn
```

<br>14. Установить веб-сервер Nginx и запустить его:
```
sudo apt install nginx -y
```
```
sudo systemctl start nginx 
```

<br>15. Открыть порты для фаерволла и активировать его:
```
sudo ufw allow 'Nginx Full'
```
```
sudo ufw allow OpenSSH
```
```
sudo ufw enable
```

<br>16. Перейти в папку infra_sprint1/frontend/, скомпилировать frontend-приложение и скопировать его в системную директорию веб-сервера (необходимо для корректной работы статических файлов):
```
npm run build
```
```
sudo cp -r <имя_пользователя>/infra_sprint1/frontend/build/. /var/www/infra_sprint1/
```

<br>17. Перейти в infra_sprint1/backend/, собрать статику для админки Django и копировать его в системную директорию веб-сервера (ВАЖНО! Необходимо изменить название папки со статикой во избежание конфликта имён):
```
infra_sprint1/backend/kittygram_backend/settings.py поменять значение переменной STATIC_URL и добавить новую STATIC_ROOT

STATIC_URL = 'static_backend'
STATIC_ROOT = BASE_DIR / 'static_backend' 
```
```
python manage.py collectstatic
```
```
sudo cp -r infra_sprint1/backend/static_backend/ /var/www/infra_sprint1/
```

<br>18. Создать папку для медиафайлов в директории веб-сервера, изменить права доступа:
```
cd /var/www/infra_sprint1/
```
```
mkdir media
```
```
sudo chown -R <имя_пользователя> /var/www/infra_sprint1/media/
```

<br>19. Отредактировать файл конфигурации Nginx и заменить его настройки следующими (требуются права root):
```
sudo nano /etc/nginx/sites-enabled/default 
```
```
server {

    server_name server_name <публичный-IP-адрес> <доменное-имя>;

    location /api/ {
        proxy_pass http://127.0.0.1:8000;
    }

    location /admin/ {
        proxy_pass http://127.0.0.1:8000;
    }

    location /media/ {
        alias /var/www/infra_sprint1/media/;
    }

    location / {
    root    /var/www/infra_sprint1;
    index   index.html index.htm;
    try_files  $uri /index.html;
    }

}
```

<br>20. Проверить файл конфигурации Nginx и перезагрузить его, если проверка прошла успешно:
```
sudo nginx -t
```
```
sudo systemctl reload nginx
```

<br>21. (Опционально) Получить SSL-сертификат для вашего доменного имени с помощью Certbot:
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

### Автор:
[Кирилл Хорошилов](https://github.com/1emd)
