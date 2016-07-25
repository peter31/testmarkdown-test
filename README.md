# Websocket chat configuration

Chat application is built on:
 - GosWebsocketBundle - https://github.com/GeniusesOfSymfony/WebSocketBundle It is used to create a websocket brigde between client and server sides.
 - Api.ai - https://api.ai/ Is used to process requests from client to messenger bot user. 
 - rabbitMQ - to send bot requests from client to api.ai service and show answer to client. 

## Chat configuration

### Project configs

It is required to set `payever.base_host` parameter to correct value in the file `app/config/parameters.private.yml`
```
payever.base_host = www.payever.local
```

If the foolowing line does not exist in `app/config/parameters.specific.yml` it is required to add (assuming)

```
payever_messenger.client_ws_url: "ws://%payever.base_host%%gos_web_socket.suffix%"
```


### Supervisor

To run chat application is required a permanent console script - `./symfony gos:websocket:server`. 
It is convenient to run that daemon script using supervisor. Configuration example:

```
#/etc/supervisor/conf.d/websocket.conf

[program:websocket]
command=/local/project/path/symfony gos:we:se
directory=/local/project/path/
autostart=true
autorestart=true
startretries=3
stderr_logfile=/var/log/supervisor/websocket.log
stdout_logfile=/var/log/supervisor/websocket.log
user=peton
#environment=
```

Start websocket supervisor daemon:
 
```
sudo supervisorctl start websocket
```

To see websocket server logs it is required to run

```
sudo supervisorctl tail -f websocket
```

Duting chat application development it is sometimes required to restart websocket daemon. It can be done by:
 
```
sudo supervisorctl restart websocket
```


### Apache vhost

It is required to add the following line in `<VirtualHost *:80>`:

```
ProxyPass "/chat/" "ws://127.0.0.1:1337"
```

It is required to enable Apache `proxy` module:if it is not done yet

```
sudo a2enmod proxy
```

### Nginx configuration

Add:
```
#websockets chat
    location /chat {
        proxy_pass http://localhost:1337;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
```

to run websocket server with `wss` you need certificates,
and nginx/apache configured to run in http/https

```
server {
        listen   443 ssl;
        server_name payever.local;

        ssl_certificate /usr/local/etc/nginx/ssl/nginx.crt;
        ssl_certificate_key /usr/local/etc/nginx/ssl/nginx.key;
        ssl on;

        rewrite     ^   http://$host$request_uri?;
}
```
