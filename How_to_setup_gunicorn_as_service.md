## How to setup gunicorn as service
### Initial setup
1. git clone .../app
2. cd app
3. python3 -m venv venv
4. source venv/bin/activate
4. pip install -r requirements.txt
5. nano app/local_settings.py
6. psql (database creation)
7. ufw allow 10000
8. python manage.py runserver 0.0.0.0:10000

### Gunicorn setup
1. gunicorn --bind 0.0.0.0:10000 app.wsgi
2. cd /etc/systemd/system
3. nano app.socket
```
[Unit]
Description=app gunicorn socket

[Socket]
ListenStream=/home/artbakulev/app/gunicorn.sock

[Install]
WantedBy=sockets.target
```
4. nano app.service
```
[Unit]
Description=app gunicorn daemon
Requires=app.socket
After=network.target

[Service]
User=sammy
Group=www-data
WorkingDirectory=/home/artbakulev/app
ExecStart=/home/app/app/venv/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/home/artbakulev/app/app.sock \
          app.wsgi:application

[Install]
WantedBy=multi-user.target
```
5. systemctl start app.socket
6. systemctl enable app.socket
7. systemctl status app.socket

### Gunicorn debug
1. file /home/artbakulev/app/gunicorn.sock
2. journalctl -u app
3. systemctl daemon-reload\


### Nginx setup
1. cd /etc/nginx/sites-available
2. nano app.com
```
server {
        server_name app.com www.app.com;

        location = /favicon.ico { access_log off; log_not_found off; }
        location /static/ {
                root /home/artbakulev/app;
        }
        location / {
                include proxy_params;
                proxy_pass http://unix:/home/artbakulev/app/app.sock;
        }
}
```
3. ln app.com ../sites-enabled
4. nginx -t
5. systemctl restart nginx

### Certbot 
1. certbot --nginx -d app.com -d www.app.com
