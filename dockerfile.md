- 若依 JAVA 代码 dockerfile

```dockerfile
FROM openjdk:8-jdk
LABEL maintainer=afumu

#docker run -e PARAMS="--server.port 9090"
ENV PARAMS="--server.port=8421"
RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai' >/etc/timezone

COPY target/*.jar /app.jar
EXPOSE 8421

#
ENTRYPOINT ["/bin/sh","-c","java -Dfile.encoding=utf8 -Djava.security.egd=file:/dev/./urandom -jar app.jar ${PARAMS}"]
```

- 若依前端代码 dockerfile

```dockerfile
# 基础镜像

FROM nginx

# author

MAINTAINER ruoyi

# 挂载目录

VOLUME /home/ruoyi/projects/ruoyi-ui

# 创建目录

RUN mkdir -p /home/ruoyi/projects/ruoyi-ui

# 指定路径

WORKDIR /home/ruoyi/projects/ruoyi-ui

# 复制conf文件到路径

COPY ./conf/nginx.conf /etc/nginx/nginx.conf

# 复制html文件到路径

COPY ./html/dist /home/ruoyi/projects/ruoyi-ui
```



- 若依 NGINX 配置文件

```nginx
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;

    server {
        listen       80;
        server_name  _;
    
        location / {
            root   /home/ruoyi/projects/ruoyi-ui;
            try_files $uri $uri/ /index.html;
            index  index.html index.htm;
        }
    
    location /prod-api/{
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $proxy_add_x_forwarded_for;
            proxy_set_header REMOTE-HOST $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-For-test $proxy_add_x_forwarded_for;
            proxy_pass http://ruoyi-be.sg-project:8421/;
        }
    
        # 避免actuator暴露
        if ($request_uri ~ "/actuator") {
            return 403;
        }
    
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }

}
```

