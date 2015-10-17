# Nginx Deployment Guide
### 1. Nginx as a revese-proxy with traffic mirroring server

Nginx의 `revese-proxy`기능과 `post_action` feature를 응용하여 UrQA HAProxy server의 traffic을 HoneyQA api server로 traffic mirroring을 구현한다.

### 2. Advantages

* UrQA의 Traffic을 HoneyQA로 `traffic mirroring`을 구현한다.
* 직접적으로 HAProxy server에 접근하지 않게 되고 nginx를 proxy로 경유하므로 `보안`의 효과가 있다.
* 부하 분산 효과

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

### 6. Deploy
