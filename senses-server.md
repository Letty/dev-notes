# Senses Additional Notes

* Projects with backend have the owner www-data (also www-data usergroup)
* new tmux session `tmux` or `tmux new -s <session_name>`
* prefix is mapped to `C-a`
* horizontal screen split `C-a -` vertical `C-a ,`

## NGINX

* multiple Flask Applications for the same domain but in different locations

```
server {
    listen 80;
    server_name <name>;

    location /random-flask-project {
        root /var/www/flask-project;
        proxy_pass http://127.0.0.1:<port>/;
        proxy_set_header Host $host;
        proxy_set_header X-Real_IP $remote_addr;
        proxy_redirect $uri http://127.0.0.1:<port>/$uri; # routing for api call
    } 
}
```

