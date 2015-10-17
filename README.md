# Nginx Deployment Guide
### 1. Nginx as a revese-proxy with traffic mirroring server

Nginx의 `revese-proxy`기능과 `post_action` feature를 응용하여 UrQA HAProxy server의 traffic을 HoneyQA api server로 traffic mirroring을 구현한다.

### 2. Advantages

* UrQA의 Traffic을 HoneyQA로 `traffic mirroring`을 구현한다.
* 직접적으로 HAProxy server에 접근하지 않게 되고 nginx를 proxy로 경유하므로 `보안`의 효과가 있다.
* 부하 분산 효과
* Static File을 NginX에 위임할 경우 보다 괜찮은 성능을 보여준다.

### 3. What is reverse-proxy?

* Reverse Proxy는 실제 요청을 처리하는 서버의 앞 단에 존재하며, 실제 서버로 들어오는 요청을 대신 받아서 해당 서버에 전달해 주고 그 결과를 받아서 요청한 곳으로 전달해주는 역할을 하는 서버를 말한다. 
* Reverse Proxy는 보안(실제 서버를 외부에 숨길 때) 및 부하 분산(여러 서버에서 요청을 처리할 때) 등의 이유로 필요하다.

### 4. What is post_action feature?

Post traffic을 mirroring할 수 있는 feature로 `proxy_pass`와 함께 사용할 수 있다.
```{.no-highlight}
* location / {
    uwsgi_pass      unix:app.sock;
    post_action @post_action; 
}

location @post_action {
    proxy_pass      http://dst_host:dst_port; 
}
```

### 5. Configurations

```{.no-highlight}
 # reverse proxy
        server {
        listen       80;
        server_name  UrQA.io www.UrQA.io;
 
        #charset
 
        #access_log  logs/host.access.log  main;
        log_not_found off;
 
        client_max_body_size    30m;
        large_client_header_buffers 4 16k;
 
        location / {
            proxy_pass  http://urqa.io;
            proxy_set_header Accept-Encoding   "";
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_redirect off;
            uwsgi_pass      unix:app.sock;
            
            post_action @post_action; 
            
        }
        
        location @post_action {

        proxy_pass      http://api2.honeyqa.io; # traffic mirroring 
        
        }
 
        #error_page  404              /404.html;
 
        # redirect server error pages to the static page /50x.html
        #
        #error_page   500 502 503 504  /50x.html;
        #location = /50x.html {
        #    root   html;
        #}
 
        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        location ~ /\.ht {
            deny  all;
        }
    }
```

```{.no-highlight}
user  www www;

worker_processes  2;

pid /var/run/nginx.pid;

#                          [ debug | info | notice | warn | error | crit ]

error_log  /var/log/nginx.error_log  debug;

events {
    connections   2000;

    # use [ kqueue | rtsig | epoll | /dev/poll | select | poll ];
    use kqueue;
}
```

```{.no-highlight}
fastcgi_param  QUERY_STRING       $query_string;
fastcgi_param  REQUEST_METHOD     $request_method;
fastcgi_param  CONTENT_TYPE       $content_type;
fastcgi_param  CONTENT_LENGTH     $content_length;

fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
fastcgi_param  REQUEST_URI        $request_uri;
fastcgi_param  DOCUMENT_URI       $document_uri;
fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi_param  SERVER_PROTOCOL    $server_protocol;
fastcgi_param  HTTPS              $https if_not_empty;

fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
fastcgi_param  SERVER_SOFTWARE    nginx/$nginx_version;

fastcgi_param  REMOTE_ADDR        $remote_addr;
fastcgi_param  REMOTE_PORT        $remote_port;
fastcgi_param  SERVER_ADDR        $server_addr;
fastcgi_param  SERVER_PORT        $server_port;
fastcgi_param  SERVER_NAME        $server_name;
```

### 6. Deploy
```{.no-highlight}
git clone https://github.com/honeyqa/nginx-deploy.git
nginx -t
/etc/init.d/nginx reload
/etc/init.d/nginx start
```
