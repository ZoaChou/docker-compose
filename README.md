# docker-compose
A project to collect dokcer-compose easy start environment

* docker network create nginx-proxy
* docker run -d -p 80:80 -p 443:443 --name nginx-proxy --net nginx-proxy --restart always -v /var/run/docker.sock:/tmp/docker.sock jwilder/nginx-proxy:alpine