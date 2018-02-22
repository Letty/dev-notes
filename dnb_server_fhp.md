# DNB Server

## DNB Database Server

* [Installing MySQL Server] (https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-16-04)
    - no remote root login. The database is read only for the application server 
* Upload database dump `scp dump.sql user@server:/path/to/dir`
* connect to mysql with user root `mysql -u root -p`
* create user for application server `create user 'user_name'@'ip_from_app_server';`
* create database `create database <name>`
* grant privilege to user `grant select on dbname.* to 'user_name'@'ip_from_app_server';`
* exit mysql shell 
* Import dump into database `mysql -u root -p -h localhost dbname < /path/to/file.sql `
* open mysql config and change `bind-address` from `127.0.0.1` to local IP
* restart mysql service

## DNB Application Server

* install nmap
* check if server can reach database server `nmap ip_address` and mysql port is open
* install and upgrade pip
* [set correct locales for python](https://stackoverflow.com/questions/14547631/python-locale-error-unsupported-locale-setting)

### NGINX
* install nginx `sudo apt-get install nginx`
* start nginx `sudo /etc/init.d/nginx start`
* configure nginx 
    - remove default settings `sudo rm /etc/nginx/sites-enabled/default`
    - add project settings `sudo touch /etc/nginx/sites-available/project_settings`
    - make a symlink to sites-enabled `sudo ln -s /etc/nginx/sites-available/project_settings /etc/nginx/sites-enabled/project_settings`
    - configure nginx `sudo nano /etc/nginx/sites-enabled/project_settings`
```
server {
    listen 80;
    server_name dnbvis.fh-potsdam.de;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real_IP $remote_addr;
    } 
}
```
* restart nginx `sudo /etc/init.d/nginx restart`

### Transfer Files
* create directory at `/var/www` named `dnbvis.fh-potsdam.de`
* copy files into folder or git pull from [github repo](prototype)
* change database host, user and password

### Virtual Env
* install `sudo pip install virtualenv`
* go to project folder `cd /var/www/dnbvis.fh-potsdam.de`
* create virtual environment `virtualenv env`
* activate virtual env `source env/bin/activate`

### Install Project Dependencies + Gunicorn
* install `sudo pip install flask gunicorn pymysql eventlet`
* run gunicorn `gunicorn server:app`
* have a look on the public ip adress if the app is running
* shut it down and make a wsgi file
```
from server import app

if __name__ == "__main__":
    app.run()
```
* run gunicorn with wsgi bindings `gunicorn --bind 127.0.0.1:8000 wsgi:app` check if everthing works
* close gunicorn end virtual environment `deactivate`

### Systemd Service 

* add systemd service file `sudo nano /etc/systemd/system/dnb.service`
* modify the settings
```
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
PIDFile=/var/www/dnbvis.fh-potsdam.de/app.pid
User=user
Group=usergroup
WorkingDirectory=/var/www/dnbvis.fh-potsdam.de
ExecStart=/usr/local/bin/gunicorn --bind 127.0.0.1:8000 --worker-class eventlet --workers 2 --timeout 60 --threads 2  wsgi:app --pid /var/www/dnbvis.fh-potsdam.de/app.pid
/dnbvis.fh-potsdam.de/app.pid
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
* safe your service and run `sudo systemctl daemon-reload` 
* let your service start automaticcaly at boot `sudo systemctl enable dnb.service`
* reboot or start service `sudo systemctl start dnb.service`
* `systemctl status dnb.service`

## Todo
* add certs via lets encrypt or DFN and change nginx

## Links
* [Adding MySQL Users with different privileges](https://dev.mysql.com/doc/refman/5.7/en/adding-users.html)