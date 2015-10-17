# Nginx Deployment Guide
### 1. Nginx as a revese-proxy with traffic mirroring server
Nginx의 `revese-proxy`기능과 `post_action` feature를 응용하여 UrQA HAProxy server의 traffic을 HoneyQA api server로 traffic mirroring을 구현한다.
### 2. Advantages

* 직접적으로 HAProxy server에 접근하지 않게 되고 nginx를 proxy로 경유하므로 `보안`의 효과가 있다.
* UrQA의 Traffic을 HoneyQA로 `traffic mirroring`을 구현한다.

### 2. What is reverse-proxy?
### 3. What is post_action feature?
### 4. Configurations
