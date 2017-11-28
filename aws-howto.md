# AWS EC2 Ubuntu Instance - Python Flask App with nginx and gunicorn

* log into ec2 ubuntu instance
* check if python is installed
* install pip for python3
```
sudo apt-get install python3-setuptools
sudo easy_install3 pip
```

## NGINX
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
    server_name domain_or_ip;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real_IP $remote_addr;
    } 
}
```
* restart nginx `sudo /etc/init.d/nginx restart`

# Transfer Files
* go to your directory on your local machine to transfer data between your machine and ec2
* `scp -r -i /path/to/key.pem /local/path/to/files user@awshost.com:~/destination/path`

## Virtual Env
* install `sudo pip install virtualenv`
* go to project folder `cd /home/ubuntu/project`
* create virtual environment `virtualenv env`
* activate virtual env `source env/bin/activate`

## Install Project Dependencies + Gunicorn
* install `sudo pip install flask gunicorn <packages..>`
* run gunicorn `gunicorn python_file:app`
* have a look on the public ip adress if the app is running
* shut it down and make a wsgi file
```
from <python_file> import <app_name>

if __name__ == "__main__":
    <app_name>.run()
```
* run gunicorn with wsgi bindings `gunicorn --bind 127.0.0.1:8000 wsgi_file:app` check if everthing works
* shut it down

## Systemd Service 

* add systemd service file for your project `sudo nano /etc/systemd/system/<yourprojectname>.service`
* modify the settings
```
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
PIDFile=/home/user/project/app.pid
User=user
Group=usergroup
WorkingDirectory=/home/user/project
ExecStart=/usr/local/bin/gunicorn --bind 127.0.0.1:8000 wsgi:app --pid /home/user/project/app.pid
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
* safe your service and run `sudo systemctl daemon-reload` 
* let your service start automaticcaly at boot `sudo systemctl enable <yourprojectname>.service`
* reboot or start service `sudo systemctl start <yourprojectname>.service`
* `systemctl status <yourprojectname>.service`

## Lets encrypt

* if you used the ec2 ip -> change the server_name field in nginx config to domain name
* follow instructions at certbot website [1]
* restart nginx
* horray!

## Improvements

* use github/gitlab with deploy key on aws

## Useful Links
[1] - https://certbot.eff.org/#ubuntuxenial-nginx
[] - https://www.youtube.com/watch?v=kDRRtPO0YPA
[] - https://stackoverflow.com/questions/44818451/setup-gunicorn-to-run-with-systemd
[] - http://bartsimons.me/gunicorn-as-a-systemd-service/
